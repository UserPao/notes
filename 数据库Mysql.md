## 如何设计关系型数据库

1. 存储

   文件系统将数据持久化到磁盘当中

2. 程序实例

   1. 存储管理：数据逻辑关系转换成物理存储关系的模块
   2. 缓存机制：优化执行效率
   3. SQL解析：对SQL语句进行解析
   4. 日志管理：记录操作
   5. 权限划分：进行多用户管理
   6. 容灾机制：灾难恢复
   7. 索引模块：优化数据查询
   8. 锁模块：使得数据库支持并发操作

## Count()的问题

- count(1) 和count(*)的区别

  使用count函数，当要统计的数量比较大时，发现count( * )花费的时间比较多，相对来说count(1)花费的时间比较少。

  1、如果你的数据表没有主键，那么count(1)比count( * )快 ；如果有主键的话，那主键（联合主键）作为count的条件也比count( * )要快  。

  2、如果你的表只有一个字段的话那count(*)就是最快的。  

  3、如果count(1)是聚索引,id,那肯定是count(1)快,但是差的很小的。因为count(*),自动会优化指定到那一个字段。所以没必要去count(1)，用count(*),sql会帮你完成优化。此时count(1)和count(*)基本没有区别! 

- 在数据记录都不为空的时候查询出来结果上没有差别的

  count(*)（是针对全表）将返回表格中所有存在的行的总数**包括值为null**的行；

  count(列名)（是针对某一列）将返回表格中**某一列除去null**以外的所有行的总数。

- 执行效率

  列名为主键，count(列名)会比count(1)快  

  列名不为主键，count(1)会比count(列名)快 
  如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*） 
  如果有主键，则 select count（主键）的执行效率是最优的 

  如果表只有一个字段，则 select count（ * ）最优

## 主键模块

### 为什么不使用uuid作为主键

1. UUID生成速率低下

   ​		Java的UUID依赖于SecureRandom.nextBytes方法，而SecureRandom又依赖于操作系统提供的随机数源，在Linux系统下，它的默认依赖是/dev/random，而这个源是阻塞的。最可怕的是，这个nextBytes方法还是一个synchronized方法，也就是说，如果多线程调用UUID，生成速率不升反降。

   测试结果：在一台64线程的服务器上，调用UUID.randomUUID方法，生成一千万个uuid平均耗时在130s，tps不到8w

2. UUID主键在innodb中会引发性能问题

   1. innodb中的主键索引也是聚集索引，如果插入的数据是顺序的，那么b+树的叶子基本都是满的，缓存也可以很好的发挥作用。如果插入的数据是完全无序的，那么叶子节点会频繁分裂，缓存也基本无效了。这会减少tps
   2. uuid占用的空间较大

3. UUID完全没有意义，如果有一个主键是全局自增的，那么数据排列顺序就是数据的插入顺序

## 索引模块

**mysql一张表最多可以创建16个索引**

### 覆盖索引

覆盖索引可以减少回表的次数，mysql5.6以后对覆盖索引做了进一步的优化，支持**索引下推**，索引下推也就是把覆盖索引所覆盖的字段进行进一步筛选，减少回表的次数。

**假如机器磁盘是机械磁盘的话，随机读写会有磁盘定位的开销，这个时候可以打开MRR【multi range read】,可以在回表之前， 把ID读到buffer中，进行排序，把随机的操作变成顺序的操作**

###  什么信息能成为索引

​	能把该记录限定在一定查找范围内的字段

​	@@ 主键、唯一键、普通键

### 索引的数据结构

1. 二叉查找树进行二分查找
2. B Tree
3. B+Tree
4. Hash结构

#### 二叉查找树

缺点：增加和删除节点可能会导致树结构样式的改变，

#### B Tree

1. 定义
   1. 根节点至少包含两个孩子
   2. 树中每个节点最多含有m个孩子（m >= 2）
   3. 除了根节点和叶结点以外，每个节点至少有ceil（m/2)【取上限】个孩子
   4. 所有叶子节点位于同一层
   5. 假设每个非终端节点中包含有n个关键字信息，其中
      1. Ki（i=1...n)为关键字，且关键字顺序按照升序排序K(i - 1) <Ki
      2. 关键字的个数n必须满足[ceil(m/2) - 1] <= n <= m-1【任一节点的关键字个数上限比孩子树上限少一个，且对于非叶子节点来说，任何一个节点的关键字的个数比他的指向孩子的节点个数少一个】
      3. 非叶子节点的指针：p[1], p[2], ..., p[M]；其中p[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M - 1]的子树，其他P[i]指向关键字属于（K[i - 1], K[i]） 的子树
2.  ![1587632655810](E:\文档\学习资料\笔记\面经\数据库Mysql.assets\1587632655810.png)
   - B树中关键字集合分布在整棵树中，叶节点中不包含任何关键字信息
   - B树中任何一个关键字只出现在一个结点中

#### B+Tree【默认使用】

1. 定义
   1. B+树是B树的变体，其定义基本与B树相同，除了：
      1. 非叶子节点的子树与关键字的个数相同（B树是关键字比子树少一个）
      2. 非叶子节点的子树指针P[i]，指向关键字值    [K[i]，K[i + 1]） 的子树
      3. 非叶子节点都是索引，具体的值都保存在叶子节点中
      4. 所有叶子节点都有一个链指针指向下一个叶子节点，并且都是按照大小顺序排列的
   
2. ![1587632703739](E:\文档\学习资料\笔记\面经\数据库Mysql.assets\1587632703739.png)

   而B+树关键字集合分布在叶子结点中，非叶节点只是叶子结点中关键字的索引；

   而B+树中的关键字必须出现在叶节点中，也可能在非叶结点中重复出现；

3. **比B树更适合做存储索引的优势**

   1. B+树的磁盘读写代价更低。B+树的**内部结点并没有指向关键字具体信息的指针**，其内部结点比B树小，盘块能容纳的结点中关键字数量更多，一次性读入内存中可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素。
   2. B+树的查询效率更加稳定，都是想通过的到达叶子节点O（lgn）
   3. B+树更有利于对数据库的范围查询【因为有叶子节点之间连接的指针】

4. **为什么不使用红黑树，而使用B+树**

   AVL 树和红黑树这些二叉树结构的数据结构可以达到最高的查询效率这是毋庸置疑的。

   既然如此，那么数据库索引为什么不用 AVL 树或者红黑树呢？

   AVL 数和红黑树基本都是存储在内存中才会使用的数据结构，那磁盘中会有什么不同呢？

   这就要牵扯到磁盘的存储原理了

   操作系统读写磁盘的基本单位是扇区，而文件系统的基本单位是簇(Cluster)。

   也就是说，磁盘读写有一个最少内容的限制，即使我们只需要这个簇上的一个字节的内容，我们也要含着泪把一整个簇上的内容读完。

   那么，现在问题就来了

   一个父节点只有 2 个子节点，并不能填满一个簇上的所有内容啊？那多余的内容岂不是要浪费了？我们怎么才能把浪费的这部分内容利用起来呢？哈哈，答案就是 B+ 树。

   由于 B+ 树分支比二叉树更多，所以相同数量的内容，B+ 树的深度更浅，深度代表什么？代表磁盘 io 次数啊！数据库设计的时候 B+ 树有多少个分支都是按照磁盘一个簇上最多能放多少节点设计的啊！

   所以，涉及到磁盘上查询的数据结构，一般都用 B+ 树


#### Hash

1. 缺点
   1. 仅仅能满足 = 和in操作，不能满足范围查询，因为hash函数得到的值并不能保证和本值一样的顺序
   2. 无法被用来避免数据的排序操作
   3. 不能利用部分索引建查询
   4. 不能避免表扫描
   5. 遇到大量Hash值相等的情况时，性能并不一定就会比BTree索引高
2. 优点
   1. 在处理较小数据量时有无法比拟的优势，适合做缓存（内存数据库）

#### BitMap（位图索引）

​	只适用于某个字段的值只有固定的数的情况，锁的粒度很大

### 密集索引和稀疏索引的区别

#### 密集索引

- 文件中的每个搜索码值都对应一个索引值
- 稀疏索引文件只为索引码的某些值建立索引
- InnoDB
  - 若一个主键被定义，该主键则作为密集索引
  - 若没有主键被定义，该表的第一个唯一非空索引则作为密集索引
  - 若不满足以上条件，innoDB内部会生成一个隐藏主键（密集索引）
- 非主键索引存储相关键位和其对应的主键值，包含两次查找
- innoDB索引和数据存储到一个文件中【.ibd】
- MYIASM索引存储在【.MYI】，数据存储在【.MYD】

### 主键索引和普通索引的区别

1. **主键索引索引着数据，然后普通索引索引着主键ID值**(这是在innodb中，但是如果是myisam中，主键索引和普通索引是没有区别的都是直接索引着数据)

2. 当你查询用的是where id=x 时，那只需要扫描一遍主键索引，然后拿到相应数据
   但是如果是查询的普通索引的话，那么会先扫描一次普通索引，拿到主键值，然后再去扫主键索引，拿到所需要的数据，这个过程叫做回表

3. **一定要回表查询么？**
   **不一定**，当查询的字段刚好是索引的字段或者索引的一部分，就可以不用回表，这也是**索引覆盖**的原理

4. **不用回表的情况**

   比如说：我们有以下表：

   ![1584783605168](E:\文档\学习资料\笔记\面经\数据库Mysql.assets\1584783605168.png)

   ~~~sql
   ## 对name字段建立索引，并且id为主键索引
   
   ## 则会进行回表，因为索引中并没有所有字段信息
   select * from table where name = ‘aa'，
   
   ## 此时就不会回表
   select name from table where name = 'aa'
   ~~~

5. **普通索引的change buffer**

   **InnoDB 的数据是按数据页为单位来读写的。当需要读一条记录的时候，并不是将这个记录本身从磁盘读出来，而是以页为单位，将其整体读入内存。在 InnoDB 中，每个数据页的大小默认是 16KB。**

   为了说明普通索引和唯一索引对更新语句性能的影响这个问题，这里先介绍一下 change buffer。

   ​		当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InnoDB 会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行 change buffer 中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。

   ​		虽然名字叫作 change buffer，实际上它是可以持久化的数据。也就是说，change buffer 在内存中有拷贝，也会被写入到磁盘上。

   ​		将 change buffer 中的操作应用到原数据页，得到最新结果的过程称为 merge。除了访问这个数据页会触发 merge 外，系统有后台线程会定期 merge。在数据库正常关闭（shutdown）的过程中，也会执行 merge 操作。

   ​		显然，如果能够将更新操作先记录在 change buffer，减少读磁盘，语句的执行速度会得到明显的提升。

   而且，数据读入内存是需要占用 buffer pool 的，所以这种方式还能够避免占用内存，提高内存利用率。

   ​		那么，**什么条件下可以使用 change buffer 呢**？

    		对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束。而这必须要将数据页读入内存才能判断。如果都已经读入到内存了，那直接更新内存会更快，就没必要使用 change buffer 了。

    		因此，唯一索引的更新就不能使用 change buffer，实际上也只有普通索引可以使用。

    		**使用普通索引并且因为change buffer可以提升内存命中率**

   ​		**因此，对于写多读少的业务来说，页面在写完以后马上被访问到的概率比较小，此时 change buffer 的使用效果最好。这种业务模型常见的就是账单类、日志类的系统。**

   ​		特别地，在使用机械硬盘时，change buffer 这个机制的收效是非常显著的。所以，当你有一个类似“历史数据”的库，并且出于成本考虑用的是机械硬盘时，那你应该特别关注这些表里的索引，尽量使用普通索引，然后把 change buffer 尽量开大，以确保这个“历史数据”表的数据写入速度。

   1. 从查询过程来说，唯一索引和普通索引几乎无差别
   2. 从更新过程来说，由于change buffer的加成，当要更新的目标页不在内存中时，普通索引因为减少随机 IO 的访问，对更新性能的提升很明显
   3. 大部分场景普通索引都优于唯一索引（在业务代码保证唯一性的前提下），但如果业务模式是更新完后马上查询，此时普通索引+change buffer会起副作用

### 调优SQL

#### 如何定位并优化慢查询sql

- 根据慢日志定位慢查询sql

  ~~~sql
  show variables like '%quer%'
  # slow_query_log  默认关闭，控制慢日志是否打开的变量
  # slow_query_log_file 记录慢日志的文件
  # long_query_time 被定义为慢sql的执行时间
  
  show status like '%slow_queries%'
  # slow_queries 慢查询sql的数量
  
  set global slow_query_log = on; # 打开慢查询日志
  set global slow_query_time = 1; # 设置慢查询时间为1秒。设置后需要从新连接客户端
  
  ~~~

  

- 使用explain等工具分析sql

- ![1584593221733](C:\Users\32091\AppData\Roaming\Typora\typora-user-images\1584593221733.png)

  ~~~sql
  explain select name from person_info_large order by name desc; # 使用explain
  # 字段分析
  1. type mysql找到数据行的方式
  index 、all表示全表扫描
  2. extra 
  出现以下两种意味着MYSQL根本不能使用索引，效率会受到很大影响，应尽可能进行优化语句
  Using filesort 
  表示MYSQL会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容，可能在内存或者磁盘上进行排序。MYSQL中无法利用索引完成的排序操作成为文件排序
  Using temporary 表示MYSQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by
  ~~~
  

  
- 修改sql或者尽量让sql走索引

#### 联合索引的最左匹配原则的成因

- 最左匹配原则

  ​		mysql会一直向右匹配直到遇到范围查询【>, <, between, like】就停止匹配，如果a = 3 and b = 4 and c > 5 and d = 6 ,如果建立(a, b, c, d)的索引，d是用不到索引的，如果建立(a, b, d, c)的索引则都可以用到，a, b, d的顺序可以任意调整

  ​		= 和 in 可以乱序，如果a = 1 and b = 2 and c = 3建立(a, b, c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

  有两列a，b，设置成联合索引（a，b）当调用where a = 1的时候走索引  ，调用where b = 1的时候不走索引，

#### 索引建立的越多越好吗

- 数据量小的表不需要建立索引，建立会增加额外的索引开销
- 数据变更需要维护索引，因此更多的索引意味着更多的维护成本
- 更多的索引意味着更多的空间

## MVCC【Multi-Version Concurrency Control]

### 特点

1.MVCC其实广泛应用于数据库技术，像Oracle,PostgreSQL等也引入了该技术，即适用范围广

2.MVCC并没有简单的使用数据库的行锁，而是使用了**行级锁，row_level_lock**,而**非InnoDB中的innodb_row_lock.**

### 基本原理

MVCC的实现，通过保存数据在某个时间点的快照来实现的。这意味着一个事务无论运行多长时间，在同一个事务里能够看到数据一致的视图。根据事务开始的时间不同，同时也意味着在同一个时刻不同事务看到的相同表里的数据可能是不同的。

### 基本特征

- 每行数据都存在一个版本，每次数据更新时都更新该版本。
- 修改时Copy出当前版本随意修改，各个事务之间无干扰。
- 保存时比较版本号，如果成功（commit），则覆盖原记录；失败则放弃copy（rollback）

### InnoDB存储引擎MVCC的实现策略

在每一行数据中额外保存两个隐藏的列：当前行创建时的版本号和删除时的版本号（可能为空，其实还有一列称为回滚指针，用于事务回滚，不在本文范畴）。这里的版本号并不是实际的时间值，而是系统版本号。每开始新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询每行记录的版本号进行比较。

每个事务又有自己的版本号，这样事务内执行CRUD操作时，就通过版本号的比较来达到数据版本控制的目的

### MVCC下InnoDB的增删改查是怎么实现的

- 增加

  记录的版本号即当前事务的版本号

  执行一条数据语句：insert into testmvcc values(1,"test");

  假设事务id为1，那么插入后的数据行如下：

  ![Mysql中MVCC的使用及原理详解](http://p98.pstatp.com/large/pgc-image/1536286392011332dc79980)

- 更新

  在更新操作的时候，采用的是先标记旧的那行记录为已删除，并且删除版本号是事务版本号，然后插入一行新的记录的方式。

  比如，针对上面那行记录，事务Id为2 要把name字段更新

  update table set name= 'new_value' where id=1;

  ![Mysql中MVCC的使用及原理详解](http://p98.pstatp.com/large/pgc-image/15362864790262a85896e55)

- 删除

  delete from table where id=1;

  ![Mysql中MVCC的使用及原理详解](http://p9.pstatp.com/large/pgc-image/15362865324150dfbc7bf66)

- 查询

  从上面的描述可以看到，在查询时要符合以下两个条件的记录才能被事务查询出来：

  1) 删除版本号未指定或者**删除版本号大于当前事务版本号**，即查询事务开启后确保读取的行未被删除。(即上述事务id为2的事务查询时，依然能读取到事务id为3所删除的数据行)

  2)**创建版本号 小于或者等于 当前事务版本号** ，就是说记录创建是在当前事务中（等于的情况）或者在当前事务启动之前的其他事物进行的insert。

### 适用场景

1.MVCC手段只适用于Mysql隔离级别中的读已提交（Read committed）和可重复读（Repeatable Read）.

## 锁模块

#### MyISAM和InnoDB关于锁方面的区别是什么

- MyISAM默认使用表级锁，不支持行级锁 

- InnoDB默认使用行级锁，也支持表级锁

- **innoDB在sql没有走索引的时候，用的是表级锁，在走索引的时候，用的是行级锁**

  **在mysql中，行级锁不是直接锁记录，而是锁索引，如果一条sql语句操作了主键索引，Mysql就会锁定这条主键索引，如果一条语句操作了非主键索引，mysql会先锁定该非主键索引，在锁定对应的主键索引**

  给select语句上排它锁

  select * from tableName for update // 这样就是上的排它锁，本来select上的是读锁  共享锁 

- MyISAM适合的场景

  - 频繁执行全表count语句
  - 对数据进行增删改的频率不高，查询的频率很高【**查询会加表锁**】
  - 没有事务

- InnoDB适合的场景

  - 增删改频率较高的，锁行，避免阻塞
  - 可靠性要求较高，支持事务

> 锁的分类

- 按照锁的粒度
  - 表级锁，行级锁，页级锁
- 按照锁的级别
  - 共享锁，排它锁
- 按照加锁的方式
  - 自动锁，显示锁【lock in share mode】
- 按照操作划分
  - DML锁【表数据】，DDL锁【表结构】
- 按照使用方式
  - 乐观锁【CAS】，悲观锁

#### 数据库事务的四大特性

- A 【原子性（Atomic）】
- C 【一致性（Consistency）】
- I  【隔离性（Isolation）】
- D【持久性（Durability）】

#### 事务隔离级别以及各级别下的并发访问问题

- 脏读 - READ-COMMITTED事务隔离级别以上可以避免

  ~~~sql
  select @@tx_isolation;  // 查看事务隔离级别，mysql默认是REPITABLE-READ
  set session transaction isolation level read uncomitted; // 设置当前隔离级别为read uncommitted 
  ~~~

- 不可重复读 -REPEATABLE-READ事务隔离级别以上可以避免

  ~~~sql
  # 在多次读取同一数据的时候会出现不同的结果
  # 解决办法 ，提升事务隔离级别，提升到 REPEATABLE-READ
  ~~~

- 幻读 -  SERIALIZABLE 事务隔离级别以上可以避免，直接加表锁 

  ~~~sql
  # 事务 A 读取与搜索条件相匹配的若干行，事务 B 以插入或者删除行的方式修改事务A的结果集，导致事务A看起来像出现幻觉一样
  ~~~

#### InnoDB可重复度读隔离级别下如何避免幻读

- 当前读和快照读

  - 当前读【加了锁的增删改查语句】，读取的是最新版本 

    ~~~sql
     select ***lock in share mode,
     select *** for update
     update
     delete 
     insert
    ~~~

  - 快照读

    ~~~sql
    不加锁的非阻塞读，select
    ~~~

-  本质是因为next-key 锁（行锁+ gap锁）

  - 行锁

    只有当查询走索引的时候，才会走行锁，

    总是会在索引上加锁，即使一个表没有设置任何索引，这时候InnoDB会创建一个隐式的聚集索引，然后在这个聚集索引上进行加锁行为的维护

  - gap锁

    锁定一个范围，但不包括记录本身，
  
    > 对主键索引或者唯一索引会用Gap锁吗
    >
    > 1. 如果where 条件全部命中，则不会用Gap锁，只会加记录锁
    > 2. 如果where条件部分命中或者全不命中，则会加Gap锁
    > 3. gap锁会用在非唯一索引【对区间上锁】或者不走索引【会对所有该列上锁】的当前读中

#### RC、RR级别下的InnoDB的非阻塞【快照读】读如何实现

	- 数据行中的DB_TRX_ID【最近一次对本行修改的事务的标识符】, DB_ROLL_PTR【回滚指针】, DB_ROW_ID【自增的隐藏字段】字段
 - undo日志
          对记录做了变更操作时，就会产生undo日志，每条`undo日志`也都有一个`roll_pointer`属性（`INSERT`操作对应的`undo日志`没有该属性，因为该记录并没有更早的版本），undo记录中存储的是老版本的数据，为了能读取到老版本的数据，要顺着undo链找到满足其可见性的记录
       
    - insert  undo log
      	- update undo log
    
- read view

    ​        对于使用READ UNCOMMITTED隔离级别的事务来说，直接读取记录的最新版本就好了，对于使用SERIALIZABLE隔离级别的事务来说，使用加锁的方式来访问记录。对于使用READ COMMITTED和REPEATABLE READ隔离级别的事务来说，就需要用到我们上边所说的版本链了，核心问题就是：需要判断一下版本链中的哪个版本是当前事务可见的。所以设计InnoDB的大叔提出了一个ReadView的概念，这个ReadView中主要包含当前系统中还有哪些活跃的读写事务，把它们的事务id放到一个列表中，我们把这个列表命名为为m_ids。这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见
    
    - 如果被访问版本的`trx_id`属性值小于`m_ids`列表中最小的事务id，表明生成该版本的事务在生成`ReadView`前已经提交，所以该版本可以被当前事务访问。
    
    - 如果被访问版本的`trx_id`属性值大于`m_ids`列表中最大的事务id，表明生成该版本的事务在生成`ReadView`后才生成，所以该版本不可以被当前事务访问。
    
      在`MySQL`中，`READ COMMITTED`和`REPEATABLE READ`隔离级别的的一个非常大的区别就是它们生成`ReadView`的时机不同，我们来看一下。
    
      **READ COMMITTED --- 每次读取数据前都生成一个ReadView**
    
      **REPEATABLE READ --- 在第一次读取数据时生成一个ReadView**
    
      
    
    可见性判断。针对查询的数据创建read view 来决定当前事务能看到哪个版本的数据，遵循可见性算法

### 语法

- GROUP BY 
- HAVING
- COUNT, SUM, MAX, MIN, AVG

## 数据库设计的六个阶段

1：需求分析：分析用户的需求，包括数据、功能和性能需求；
2：概念结构设计：主要采用E-R模型进行设计，包括画E-R图；
3：逻辑结构设计：通过将E-R图转换成表，实现从E-R模型到关系模型的转换；
4：数据库物理设计：主要是为所设计的数据库选择合适的存储结构和存取路径；
5：数据库的实施：包括编程、测试和试运行；
6：数据库运行与维护：系统的运行与数据库的日常维护。

## 红黑树

- 特点
  - 每个节点不是红的就是黑的
  - 根节点是黑的
  - 每个叶结点都是黑色的空节点
  - 如果节点是红色的，那么他的子节点必须是黑色的
  - 从根节点到叶结点或空节点的每条路径，必须包含相同数目的黑色节点
  - 红黑树的应用
    - TreeMap，TreeSet以及JDK8以后的HashMap
  - 作用
    - 为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构

## B-， B+， B* 树

B树是一种平衡的多路查找树，

B+ 树的叶子节点链表结构相比于B-树更便于扫库和范围检索

B+树支持range-query，B树不支持，这就是数据库选B+树的主要原因

B* 树是B+树的变体，B*树分配新节点的概率要比B+树低，空间使用率更高

## LSM树

B+ 树最大的性能问题是会产生大量随机的IO

为了克服B+树的弱点，HBase引入了LSM树【Log-Structured-Merge-Trees】

## 存储过程

~~~sql
1、建立存储过程完成图书管理系统中的借书功能。
   功能要求：                  
l    借书时要求输入借阅流水号，借书证号，图书编号。（即该存储过程有3个输入参数）                        
l    借书时，借书日期为系统时间。
l    图书的是否借出改为‘是’  

create or replace procedure lendBook
(
 v_lendid  lend.lid%type,
 v_readerid  reader.rid%type,
 v_bookid book.bid%type
)
as
begin
     insert into lend values(v_lendid, v_readerid, v_bookid,SYSDATE, NULL, NULL, NULL);
     update book set b_isborrow= '是'
     where book.bid = v_bid;
end;

## 调用方式
call lendBook(1, 1, 1);
~~~

## 数据库连接池

~~~java
package MyDruid;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.LinkedList;
import java.util.logging.Logger;

/**
 * @Author: huki
 * @Date: 2020/4/7 14:48
 * @Version 1.0
 */
public class MyDataSource implements DataSource {
    /**
     * 链表 --- 实现栈结构
     */
    private LinkedList<Connection> dataSources = new LinkedList<>();

    public MyDataSource(){
        for(int i = 0 ; i < 10 ; i++ ){
            try{
            // 1. 装载sqlserver驱动
                DriverManager.registerDriver(new SQLServerDriver());
                //2、通过JDBC建立数据库连接
                Connection con =DriverManager.getConnection(
                        "jdbc:sqlserver://192.168.2.6:1433;DatabaseName=customer", "sa", "123");
                //3、将连接加入连接池中
                dataSources.add(con);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    @Override
    public Connection getConnection() throws SQLException {
        // 删除第一个连接返回
        final Connection conn = dataSources.removeFirst();
        return conn;
    }
    /**
     * 将连接放回连接池
     */

    public void releaseConnection(Connection conn) {
        dataSources.add(conn);
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {

    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return null;
    }
}

~~~

## sql注入

~~~sql
select * from users where userName = '' or 1 = 1# ' and password=md5('') 
相当于
select* from users where usrername='' or 1=1   #后边是注释  相当于屏蔽了
~~~

## Mybatis

### #{}和${}的区别

使用**#{}**的时候，sql语句解析的时候会给变量加上“”双引号，

可以方式**sql注入**

~~~sql
select * from user where name = #{name} 
当name为小李的时候,解析结果是
select * from user where name = "小李"
~~~

**当需要进行动态排序的时候 必须使用${}**

~~~java
select * from table order by #{name} 
当name为age的时候  就会变成
select * from table order by 'age' 这是不对的
select * from table order by ${name}
当name为age的时候  就会变成
select * from table order by age 这是对的
~~~

**两者的区别**

1. #将传入的数据都当成一个**字符串**，会对自动传入的数据加一个双引号。如：order by #user_id#，如果传入的值是111,那么解析成sql时的值为order by "111", 如果传入的值是id，则解析成的sql为order by "id".
2. $将传入的数据直接显示生成在sql中。如：order by $user_id$，如果传入的值是111,那么解析成sql时的值为order by user_id, 如果传入的值是id，则解析成的sql为order by id.
3. #方式能够很大程度防止sql注入。　
4. $方式无法防止Sql注入。
5. $方式一般用于传入数据库对象，例如传入表名.　
6. 一般能用#的就别用$.
7. MyBatis排序时使用order by 动态参数时需要注意，用$而不是#

### 转义符号

|  符号   |  转义结果  |
| :-----: | :--------: |
| 大于号> |   & l t;   |
| 小于号< |   & g t;   |
|   与&   |  & a m p;  |
| 单引号' | & a p o s; |
| 双引号" | & q u o t; |

### 动态标签

-  **if 标签**

  if 标签通常用于 WHERE 语句、UPDATE 语句、INSERT 语句中，通过判断参数值来决定是否使用某个查询条件、判断是否更新某一个字段、判断是否插入某个字段的值。

  ~~~xml
  <if test="name != null and name != ''">
      and NAME = #{name}
  </if>
  ~~~

- **foreach 标签**

  foreach 标签主要用于构建 in 条件，可在 sql 中对集合进行迭代。也常用到批量删除、添加等操作中。

  ~~~xml
  <!-- in查询所有，不分页 -->
  <select id="selectIn" resultMap="BaseResultMap">
      select name,hobby from student where id in
      <foreach item="item" index="index" collection="list" open="(" separator="," close=")">
          #{item}
      </foreach>
  </select>
  
  ~~~

  属性介绍：

  - collection：collection 属性的值有三个分别是 **list、array、map** 三种，分别对应的参数类型为：**List、数组、map 集合。**
  - item ：表示在迭代过程中每一个元素的别名
  - index ：表示在迭代过程中每次迭代到的位置（下标）
  - open ：前缀
  - close ：后缀
  - separator ：分隔符，表示迭代时每个元素之间以什么分隔

- **choose标签**

  ​		有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。MyBatis 提供了 choose 元素，按顺序判断 when 中的条件出否成立，如果有一个成立，则 choose 结束。当 choose 中所有 when
  的条件都不满则时，则执行 otherwise 中的 sql。类似于 Java 的 switch 语句，choose 为 switch，when 为 case，otherwise 则为 default。
  ​    	if 是与(and)的关系，而 choose 是或（or）的关系。

  ~~~xml
  <select id="getStudentListChoose" parameterType="Student" resultMap="BaseResultMap">
      SELECT * from STUDENT WHERE 1=1
      <where>
          <choose>
              <when test="Name!=null and student!='' ">
                  AND name LIKE CONCAT(CONCAT('%', #{student}),'%')
              </when>
              <when test="hobby!= null and hobby!= '' ">
                  AND hobby = #{hobby}
              </when>
              <otherwise>
                  AND AGE = 15
              </otherwise>
          </choose>
      </where>
  </select>
  ~~~

  

