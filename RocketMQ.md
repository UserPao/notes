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

