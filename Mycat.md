## 主要功能

- 分布式数据库系统中间层

- 实现数据库的读写分离

- 读数据的负载均衡

- 支持后端Mysql的高可用
  - 实现原理
    - 配置的时候可以除了指定主节点之外，还可以指定一个从节点，也支持高可用，
    - 但是并不支持子节点对新的主节点的同步
- 数据库的垂直拆分
  - 将数据按照模块进行划分，比如用户库，订单库
- 数据库的水平拆分
  - 将同一个库进行划分数据库，比如将用户库分为用户1库，用户2库

## 应用场景

- 需要进行读写分离的场景
  - 在往一台服务器上进行读写混合操作的时候，感觉到数据库的性能大幅度下降，并且读负载明显高于写负载的时候
  - mycat支持多种后端的mysql集群方案来进行读写分离，例如一主一从，一主多从
- 需要分库分表的场景
- 多租户的场景
- 数据统计系统
- HBase的替代方案
  - HBase是一款基于hadoop的分布式内存数据库，常用的关系型数据库如mysql都是行存储，可以利用垂直或者水平切分，进行分布式存储
- 需要使用同样的方式查询多种数据库的场景

## 优势

- 基于阿里的Cobar系统开发
- 开发社区活跃
- 完全开源支持自定义
- 支持多种关系型以及NOSQL数据库
- 基于JAVA开发，可以部署在多种系统中

## 概念

- 逻辑库
  - 对于外界来说是一个DB_USER  其实内部是user1，user2...
- 逻辑表
  - 对于外界来说是一个User表，其实内部是user1，user2...
- 关键特性
  - 支持多种Mysql集群
  - 支持JDBC连接nosql数据库
  - **支持自动故障切换，高可用性**，
  - 支持读写分离
  - 支持全局表
  - 支持独有的基于ER关系的分片策略
  - 支持一致性hash分片
  - 多平台支持，部署简单方便
  - 支持全局序列号

## Mycat配置

- schema.xml用于配置逻辑库以及数据节点

  ~~~xml
  <Schema><table></table></Schema> 定义逻辑库和表
  <dataNode></dataNode> 定义数据节点
  <dataHost></dataHost> 定义数据节点的物理数据源
  ~~~

- rule.xml用于配置表的分片规则

  ~~~xml
  <tableRule name = ""></tableRule> 定义表使用的分片规则
  <function  name = ""></function>  定义分片算法
  ~~~

- server.xml用于配置服务器权限

  ~~~xml
  <system><property name = ""></property></system> 定义系统配置
  <user></user> 定义连接MyCAT的用户，这里定义的用户和后端实际的用户不同，可以相同也可不同，只用在server.xml文件中存在的用户才能通过mycat使用到后端数据库
  ~~~

## Mycat配置日志级别

~~~xml
<asyncRoot level = 'info' includeLocation = "true"/>
All < Trace < Debug < Info < Warn < Error < Fatal < OFF 
一般设置到Info就够了
~~~



![](E:\文档\学习资料\笔记\面经\Mycat.assets\1585034182480.png)