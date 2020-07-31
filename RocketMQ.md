# 设计理念

**基于主题的发布订阅模式**，核心功能包括**消息发送，消息存储，消息存储**。

nameServer设计极其简单，抛弃了Zookeeper作为信息管理的注册中心，因为Topic路由信息无须在集群之间保持强一致。可以容忍**分钟级别的不一致**，**nameServer之间互不通信**，降低了实现复杂度和网络要求。但提升了性能。

高效的IO存储机制，**RocketMQ的消息存储文件设计成文件组的概念，组内单个文件大小固定，方便引入内存映射机制，所有主题的消息存储基于顺序写，同时为了兼顾消息消费和消息查找，引入了消息消费队列文件与索引文件。**

在消费投递语义中，**RocketMQ设计上保证消息一定被消费，允许重复消费，**重复消费问题由消费者在消息消费时实现幂等。

# 设计目标

1. 架构模式

   采用**发布-订阅**模式。参与的组件主要包括**消息发送者，消息服务器，消息消费，路由发现**

2. 顺序消息

   消息消费者按照消息达到的顺序消费，严格保证消息有序

3. 消息过滤

   在同一主题的消息可以按照规则只消费自己感兴趣的消息，支持**服务端**和**消费端**的过滤

   1. **消息在Broker端过滤**，Broker只讲消息消费者感兴趣的消息发送给消费者
   2. **消息在消费者端过滤**，消息过滤方式完全由消费者自定义，**缺点是很多无用的消息会从Broker传输到消费端**

4. **消息存储**

   对消息存储有两个维度的考量，**消息堆积能力**，**消息存储能力**

   RocketMQ追求消息存储的高性能，引入**内存映射机制**，**所有主题的消息顺序存储在同一个文件中**，为了避免消息无限在消息存储服务器中积累，引入了**消息文件过期机制**和**文件存储空间报警机制**

5. **消息高可用性**

   通常影响消息可靠性的有以下几种情况

   1. Broker正常关机
   2. Broker异常Crash
   3. OS Crash
   4. 机器断电，但是能立即恢复供电
   5. 机器无法开机
   6. 硬盘损坏

   对于1~4的情况，RocketMQ在**同步刷盘机制**写可以确保不丢失消息，在**异步刷盘模式**下，会丢失少量消息，对于情况5和6，一旦发生，该节点上的消息全部丢失，如果开启了**异步复制机制**，保证只丢失少量消息

6. 消息到达（消费）低延迟

   在消息不发生消息堆积时，以**长轮询**模式实现准实时的消息推送模式

7. 确保消息必须被消费一次

   通过**消费确认机制**保证

8. 回溯消息

   消费者已经消费成功的消息，由于业务要求需要重新消费，**支持按照时间回溯消息**，时间维度可以精确到秒，可以**向前或向后回溯**

9. 消息堆积

   消息存储使用磁盘文件（内存映射机制），并且**在物理布局上为多个大小相同的文件组成逻辑文件组，可以无限使用**，提供了默认三天的**过期机制**

10. 定时消息

    定时消息发送到Broker后，不会被立即消费，等到特定的时间点才消费。RocketMQ不支持任意进度的定时消费，只支持**特定延迟级别**

11. 消息重试机制

    消息在消费时，如果异常，支持消息重新投递。

# 核心目录

1. Broker
2. filter
3. filtersrv
4. namesrv

# 路由中心NameServer

## 设计思想

为了避免消息服务器的单点故障导致的整个系统瘫痪，通常会部署多台消息服务器共同承担消息的存储。

**消息生产者如何知道消息要发往哪台消息服务器**

**如果某一台消息服务器宕机，生产者如何在不重启服务的情况下感知**

**Broker消息服务器在启动时向所有NameServer注册，消息生产者在发送消息之前先从NameServer获取Broker服务器地址列表，然后根据负载算法从列表中选择一台发送消息，NameServer与Broker保持长连接，并间隔30s检测Broker是否存活。如果检测到Broker宕机,则从路由注册表中将其移除.但是路由变化不会马上通知消息生产者,降低了NameServer实现的复杂性，在消息发送端提供容错机制来保证消息发送的高可用**

**NameServer彼此之前互不通信**

## NameServer启动流程

启动类：**NameSrvStartUp**

1. 首先解析配置文件，填充**NameServerConfig，NettyServerConfig属性值**

   参数来源有两种

   1. **-c configFile通过-c命令指定配置文件的路径**
   2. 使用 -- 属性名 属性值，例如**--listenPort 9876**

2. 根据启动属性创建**NameSrvController**实例，并初始化该实例，**NameSrvController**实例为NameServer核心控制器

   创建KV配置，创建NettyServer网络处理对象，然后开启两个定时任务，在RocketMQ中此类定时任务统称为**心跳检测**

   1. **NameServer每隔10s扫描一次Broker，移除处于未激活状态的Broker**
   2. **nameServer每隔10分钟打印一次KV配置**

3. 注册JVM钩子函数并启动服务器，监听Broker，消息生产者的网络请求

   **一种常用的编程技巧**

   - **如果代码中使用了线程池，一种优雅的停机方式是注册一个JVM钩子，在JVM进程关闭之前，现将线程池关闭，释放资源**

## NameServer路由注册，故障剔除

**路由元信息**

路由实现类：**RouterInfoManager**

NameServer存储信息：

- topicQueueTable：HashMap<String  /* topic */, List< QueueData >>

  Topic消息队列路由信息，消息发送时根据路由表记性负载均衡

- brokerAddrTable：HashMap<String  /*brokerName */,  BrokerData>

  Broker基本信息，包括名称，所属集群名称，主备Broker地址

- clusterAddrTable：HashMap<String  /* clusterName  * /, Set< String  /* brokerName */>>

  Broker集群信息，存储集群中所有Broker的名称

- **brokerLiveTable**: HashMap<String  /* borkerAddr  * /, BrokerLiveInfo>

  Broker状态信息，NameServer每次收到心跳包时会替换该信息

- filterServerTable：HashMap<String  /* borkerAddr  * /, List< String > / * Filter Server */>

  Broker上的FilterServer列表，用于类模式消息过滤

> RocketMQ基于订阅发布机制，一个Topic拥有多个消息队列，一个Broker为每一个主题默认创建4个读队列4个写列队。多个Broker形成一个集群，BorkerName由相同的多台Broker组成Master-Slave架构，brokerId为0代表Master，大于0表示Slave，BrokerLiveInfo中的lastUpdateTimestamp存储上次收到Broker心跳包的时间

**路由注册**

Broker路由注册是通过Broker与NameServer的心跳功能实现的，Broker启动的时候向集群中所有的NameServer发送心跳语句，**每隔30s向集群中所有NameServer发送心跳包**，NameServer收到后更新brokerLiveTable中BrokerLiveInfo的lastUpdateTimestamp，然后**NameServer每隔10s扫描brokerliveTable，如果连续120s没有心跳，移除该Broker路由信息并关闭Socket连接**

1. Broker发送心跳包

   遍历NameServer列表，Broker消息服务器依次向NameServer发送心跳包。

   封装请求包头

   - brokerAddr：broker地址
   - brokerId：0表示Master，1表示Slave
   - brokerName
   - clusterName：集群名称
   - haServerAddr：master地址，初次请求时该值为空，Slave向NameServer注册后返回
   - requestBody
     - fiterServerList：消息过滤服务器列表
     - topicConfigWrapper：主题配置

2. 处理心跳包

   1. 路由注册需要加写锁，防止并发修改RouterInfoManager中的路由表。

      首先判断Broker所属集群是否存在，如不存在，则创建，然后将Broker名加入到集群Broker集合中

   2. 维护BrokerData信息

   3. 如果**Broker是Master，并且Broker Topic配置信息发生变化或者是初次注册，则需要创建或更新Topic路由元数据，填充topicQueueTable，其实就是为默认主题自动注册路由信息**

   4. 更新BrokerLiveInfo

   5. 注册Broker的过滤器Server地址列表，一个Broker上会关联多个FilterServer消息过滤服务器。

**路由删除**

Broker每隔30s向NameServer发送心跳包，但是如果Broker宕机了，NameServer收不到心跳包，**NameServer每隔10s扫描brokerLiveTable状态表，如果BrokerLive的lastUpdateTImestamp距离现在超过了120s。则认为Broker失效，移除该Broker，关闭Socket连接，更新topicQueueTable，brokerAddrTable，brokerLiveTable，filterServerTable**

RocketMQ有两个出发点触发路由删除

- 扫描超过120没有心跳进行删除
- Broker正常关闭，执行unregisterBroker指令

**路由发现**

RocketMQ路由发现是非实时的，因为Topic路由发生变化后，NameServer不会主动推送给客户端，而是**由客户端定时拉取主题最新的路由**

路由发现类：**DefaultRequestProccessor#getRouterInfoByTopic**

1. 调用RouterInfoManager的方法，从路由表topicQueueTable，brokerAddrTable，filterServerTable分别填充TopicRouterDate中的List< QueueData >,List< BrokerData > 和filterServer列表
2. 如果找到主题对应的路由信息并且该主题为顺序消息，则从NameServer KVConfig中获取关于顺序消息相关的配置填充路由信息，如果找不到返回TOPIC_NOT_EXISTS

## 坑

**NameServer至少要120s才能将失效Broker从路由表中移除，在故障期间，生产者Productor根据主题获取到的路由信息有宕机的Broker，导致消息发送失败**

# RocketMQ消息发送

普通消息发送有三种实现方式：

1. **可靠同步发送**

   发送者向MQ执行发送消息API，同步等待，直到消息服务返回发送结果

2. **可靠异步发送**

   发送者向MQ执行发送消息API,指定消息发送成功后的回调函数，然后立即返回，消息发送者线程不阻塞。直到运行结束，**消息发送成功或者失败的回调任务在一个新的线程中执行**

3. **单向发送**

   发送者向MQ执行发送消息API，立即返回。只管发送，不在乎消息是否成功存储在了消息服务器上

## 消息体

~~~java
// 所属主题
private String topic;
// 消息Flag(RocketMQ不做处理)
private int flag;
// 扩展属性
private Map properties;
// 消息体
private byte[];
~~~

扩展属性**properties**包括

- tag：消息TAG，用于消息过滤
- keys：Message索引建，多个用空格隔开，RocketMQ可以根据这些key快速检索到消息
- waitStoreMsgOK：消息发送时是否等消息存储完成后在返回
- delayTimeLevel：消息延迟级别，用于定时消息或者消息重试

## 生产者启动流程

消息生产者实现类**DefaultMQProducer**

~~~java
// 创建主题
void createTopic(String key, String newTopic, int queueNum, int topicSysFlag)
// key :目前未实际使用，可以和newTopic相同
// newTopic：主题名称
// queueNum：队列数量
// topicSysFlag：主题系统标签，默认为0
~~~

**消息生产者启动流程**

1. 检查productGroup是否符合要求，并改变生产者的instanceName为进程ID

2. 创建MQClientInstance实例

   整个JVM实例中只有一个MQClientInstance实例，维护一个MQClientInstance缓存表ConcurrentMap<String /* clientId */ , MQClientInstance> factoryTable = new ConcurrentHashMap<String, MQClientInstance>(),也就是同一个clientID只会创建一个MQClientInstance。

   **clientId为客户端IP+instance+（unitnma可选），当同一台物理服务器部署两个应用程序。clientID会相同**

   - 如果instance为默认值，RocketMQ会自动将instance设置为进程ID，这样避免了不同进程之间的影响，但同一个JVM中的不同消费者和不同生产者在启动时获取到的MQClientInstance实例是同一个。

3. 向MQClientInstance注册，将当前生产者加入到MQClientInstance管理中，方便后续调用网络请求，进行心跳检测

4. 启动MQClientInstance，如果MQClientInstance已经启动，则本次启动不会真正执行，

##消息发送基本流程

**验证消息，查找路由，消息发送（异常处理机制）**

默认是以同步方式发送，默认超时时间为3s

1. 消息长度验证

   主题名称和消息体不能为空，消息长度不能为0并且默认不能超过允许最大的长度4M

2. **查找主题路由信息**

   **需要主题的路由信息，才能知道消息发送到哪个Broker节点。**

   tryToFindTopicPublishInfo是查找主题的路由信息的方法。如果生产者缓存了topic的路由信息，如果该路由信息中包含了消息队列，则直接返回该路由信息。**如果没有缓存或者没有包含消息队列，则向nameServer查询该topic的路由信息**.

   **第一次发送消息时,本地没有缓存topic的路由信息,查询NameServer尝试获取，如果未找到，再次尝试使用默认主题DefaultMQProducerImpl#createTopicKey去查询，如果为true，则返回路由信息，否者抛出异常**

3. **选择消息队列**

   根据路由信息选择消费队列，返回的消费队列按照broker，序号排列。

   首先消息发送端采用重试机制，由retryTimesWhenSendFailed指定同步方式重试次数，异步重试机制在收到消息发送结构后执行回调函数之前进行重试。

   接下来就是**循环执行，选择消息队列，发送消息，发送成功则返回，异常就重试。选择消息队列有两种方式

   - sendLatencyFaultEnable = false **默认不启用Broker故障延迟机制**
   - sendLatencyFaultEnable = true 启用Broker故障延迟机制

   1. **默认机制**

      ​		首先在一次消息发送过程中，可能会多次执行选择消息队列的这个方法，lastBrokerName就是上一次选择的执行发送消息失败的Broker，第一次选择消息队列的时候为null，此时直接使用sendWichQueue自增再获取值，与当前路由表中消息队列个数取模，返回该位置的MessageQueue，如果在失败，则下次选择消息队列的时候规避上次的MessageQueue所在的Broker。

      ​		该算法在一次消息发送过程中能成功规避故障的Broker，**但如果Broker宕机，由于路由算法中的消息队列是按照Broker进行排序的，如果上一次根据路由算法选择的是宕机的Broker的第一个队列，name下次选择的是宕机的第二个队列，消息发送很可能还是失败**，**这时候就需要一种在一次消息发送失败后，暂时将该Broker排除在消息队列选择的范围外**，于是产生了故障延迟机制

   2. **Broker故障延迟机制**

      1. 首先对消息队列进行轮询获取一个消息队列
      2. 验证该消息队列是否可用
      3. 如果返回的messageQueue可用，移除latencyFaultTolerance关于该topic条目，表明该Broker故障已经恢复

      **MQFaukltStrategy**：消息失败策略，延迟实现的门面类

      1. long[] latencyMax = {50L, 100L, 550L, 1000L, 2000L, 3000L, 15000L}
      2. long[] notAvailableDruation = {0L, 0L, 30000L, 60000L, 120000L, 180000L, 600000L}

      **latencyMax 根据currentLatency（消息发送故障延迟时间）本次消息发送延迟，从latencyMax尾部向前找到第一个比currentLatency小的索引index，如果没有找到，返回0，然后根据这个index从notAvailable数组中区队对应的时间，在这个时长内，Broker将设置为不可用**

      

   **消息发送**

   **DefaultMQProducerImpl#sendKernelImpl**：消息发送API核心入口

   ~~~java
   private sendResult sendKernelImpl(final Message msg, final MessageQueue mq, final CommunicationMode communicateionMode, final SendCallback sendCallBack, final TopicPublishInfo topicPublishInfo, final long timeout)
   // msg 待发送消息
   // mq  消息将发送到该消息队列上
   // communicationMode 消息发送模式  SYNC ASYNC ONEWAY
   // sendCallback  异步消息回调函数
   // topicPublishInfo 主题路由信息
   // timeout  消息发送超时时间
   ~~~

   1. 根据MessageQueue获取Broker的网路地址

      如果MQClientInstance的brokerAddrTable中没有缓存该Broker的信息，则从NameServer中主动更新一下，如果更新后找不到，则抛出异常

   2. 为消息分配全局唯一的ID

      如果消息体默认超过4K，会对消息体进项压缩，并设置消息的系统标记为MessageSysFlag.COMPRESSED_FLAG。如果是事务prepared消息，则设置消息的系统标记为MessageSysFlag.TRANSACTION_PREPARED_TYPE

   3. 如果注册了消息发送的钩子函数，则执行消息发送之前的增强逻辑
      通过DefaultMQProducerImpl#registerSendMessageHook注册钩子处理类，并且可以注册多个

   4. 构建消息发送请求包

      主要包括生产者组，主题名称，默认创建主题key，该主题在单个Broker默认队列数，队列ID，消息系统标记，消息发送时间，消息标记，消息扩展属性，消息重试次数，是否是批量消息。

   5. 根据消息发送方式，同步、异步、单向方式进行网络传输。

      1. 同步方式

         1. 检查消息发送是否合理
            1. 检查该Broker是否有写权限
            2. 检查该Topic是否可以发送，对于默认主题不能发送消息，仅进行路由查询
            3. 检查队列合法性
         2. 如果消息重试次数超过允许的最大次数，消息进入**DLD延迟队列**，延迟队列主题：**%DLQ%+消费组名**
         3. 调用DefaultMessageStore#putMessage进行消息存储。

      2. 异步方式

         RocketMQ对消息发送的异步信息进行了并发控制，如果出现网络异常，网络超时，不会进行重试

      3. 单向发送

         只发送，不关心是否收到以及结果，没有重试机制

   6. 如果注册了钩子函数，执行after逻辑。（**就算消息发送过程中出现异常也会执行**）

## 批量发送消息

批量发送是将同一主题的多条信息一起打包发送到消息服务器，减少网络调用次数，提高网络传输效率

如果单条消息内容比较长，则打包多条消息发送回影响其他线程发送消息的响应时间。

**批量发送需要解决的问题是如何将这些消息编码以便服务器端能够正确解码出每条消息的消息内容**

~~~java
private int code; // 请求命令编码，请求命令的类型
private int version;
private int opaque;  // 客户端请求序号
private int flag; // 标记，倒数第一位表示请求类型：0 请求， 1返回，倒数第二位 1：表示oneway
private String remark; // 描述
private HashMap extFields; // 扩展属性
private transient CommandCustomHeader customHeader; // 每个请求对应的请求头信息
private transitent byte[] body; // 消息体
~~~

批量消息发送时，消息体的内容也存放在body中，**RocketMQ对单条消息内容采用固定格式进行存储**

# RocketMQ消息存储

**概要设计**

存储的文件包括**ComitLog文件、ConsumeQueue文件、IndexFile文件**，**所有主题的消息存储在同一个文件中，**确保消息发送时顺序写文件，**最大能力的确保消息发送的高性能和高吞吐**。但由于消息中间件一般是基于消息主题的订阅，这样设计给按照消息主题检索消息带来了极大的不方便。RocketMQ引入了**ConsumeQueue消息队列文件**，每个消息主题包含多个消息消费队列，每个消息队列有一个消息文件。**IndexFile索引文件**为了加速消息的索引性能，根据消息的属性快速从CommitLog文件中检索消息。

## 消息发送存储流程

1. 如果当前Broker停止工作或者Broker为Slaver角色或当前Rocket不支持写入则拒绝消息写入。
2. 如果消息的延迟级别大于0，将消息的原主题名称与原消息队列ID存入消息属性中，用延迟消息主题SCHEDULE_TOPIC、消息队列ID更新原来消息的主题与队列。
3. 获取当前可以写入的CommitLog文件
4. 在写入commitLog之前，先申请putMessageLock，也就是将消息存储到CommitLog文件是串行的
5. 设置消息的存储时间
6. 将消息追加到MapppedFile中，首先获取MappedFuke当前写指针，如果currentPos大于或者等于文件大小表明文件已经写满了。如果小于文件大小，通过slice()方法创建一个与MappedFile的共享内存区，并设置position为当前指针
7. 创建一个**全局唯一的ID**，消息ID有16个字节
8. 获取该消息在消息队列的偏移量。CommitLog保存了当前所有消息队列的当前待写入偏移量。
9. 根据消息体的长度，主题的长度，属性的长度结合消息存储格式计算消息的总长度。
10. 如果消息长度+END_FILE_MIN_BLANK_LENGTH大于commitLog文件的剩余空间，Broker会重新创建一个新的CommitLog来存储该信息，
11. 将消息内容存储在ByteBuffer中，然后创建AppendMessageResult。只是将消息存储在MappedFile对应的内存映射Buffer中，并没有刷到磁盘中。
12. 更新消息队列逻辑偏移量
13. 处理完消息追加逻辑后将释放putMessageLock锁
14. DefaultAppendMessageCallback#doAppend**只是将消息追加到内存中，需要根据是同步刷盘还是一部刷盘方式，将内存中的数据持久化到磁盘。然后执行HA主从同步复制。**

