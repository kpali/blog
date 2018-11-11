---
layout: post
title: ZooKeeper简明教程
description: "关于ZooKeeper ZooKeeper是一个开放源代码的分布式协调服务，最初是作为Apache Hadoop的一个子项目，是Google Chubby的开源实现。ZooKeeper的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。 ZooKeeper是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于ZooKeeper实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。"
category: 技术
tags: ZooKeeper
---

## 关于ZooKeeper

ZooKeeper是一个开放源代码的分布式协调服务，最初是作为Apache Hadoop的一个子项目，是Google Chubby的开源实现。ZooKeeper的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

ZooKeeper是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于ZooKeeper实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

## ZooKeeper的基本概念

### 一致性算法

说到一致性算法就不得不提Paxos算法，Paxos算法是莱斯利·兰伯特（Lesile Lamport）于1990年提出来的一种基于消息传递且具有高度容错特性的一致性算法。但是这个算法太过于晦涩，所以，一直以来都处于理论上的论文性质的东西，直到Google Chubby对Paxos算法进行了工程实践，Paxos才进入到工程界的视野中。

ZooKeeper并没有完全采用Paxos算法，而是使用了一种称为ZooKeeper Atomic Broadcast（ZAB，ZooKeeper原子消息广播协议）的协议作为其数据一致性的核心算法。ZAB协议并不像Paxos算法那样通用的分布式一致性算法，它是特别为ZooKeeper设计的崩溃可恢复的原子消息广播算法。

ZAB协议的整个过程大概如下：

1. 发现阶段（Discovery）：

   主要就是Leader选举过程，当整个服务器框架在启动过程中，或当Leader服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB协议就会进入恢复模式并选举产生新的Leader服务器。

2. 同步阶段（Synchronization）：

   当选举产生了新的Leader服务器后，就会进入数据同步阶段，保证集群中存在过半的机器能够和Leader服务器的数据状态保持一致。有过半的机器与Leader服务器完成数据同步后，ZAB协议就会退出恢复模式。

3. 广播阶段（Broadcast）：

   ZAB协议正式开始接收客户端新的事务请求，Leader服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议。如果集群中的其他机器接收到客户端的事务请求，会将这个事务请求转发给Leader服务器。

   当一台同样遵守ZAB协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个Leader服务器在负责进行消息广播，那么新加入的服务器就会自觉地进入数据恢复模式：找到Leader所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

### 集群角色

- Leader（领导者）

  - 由集群中所有机器通过选举产生，同一时刻只有一个Leader。
  - 为客户端提供读和写服务。

  - 事务请求的唯一调度和处理者，保证集群事务处理的顺序性。
  - 集群内部各服务器的调度者。

- Follower（跟随者）

  - 只提供读服务。
  - 处理客户端非事务请求，转发事务请求给Leader服务器。
  - 参与事务请求Proposal的投票。
  - 参与Leader选举的投票。

- Observer（观察者）

  - 只提供读服务。
  - 与Follower的唯一区别在于Observer机器不参与Leader的选举过程，也不参与写操作的“过半写成功”策略，因此Observer机器可以在不影响写性能的情况下提升集群的读性能。

### 会话（Session）

- 客户端与服务器第一次TCP连接建立开始，客户端会话的生命周期开始
- sessionTimeout值用来设置一个客户端会话的超时时间，在客户端连接断开时，只要在sessionTimeout规定的时间内重新连接集群，会话仍然有效。

### 数据节点（ZNode）

- ZooKeeper将数据存储在内存中，数据模型是一棵树（ZNode Tree），与UNIX文件系统类似，一个路径就是一个ZNode，例如`/foo/path1`
- ZNode分为持久节点和临时节点两类。
- 持久节点被创建后，除非主动进行移除，否则一直保存在ZooKeeper上。
- 临时节点的生命周期与客户端会话绑定，一旦客户端会话失效，这个客户端创建的所有临时节点都会被移除。
- ZooKeeper允许用户为每个节点添加一个特殊的属性`SEQUENTIAL`，标记该属性的节点被创建时，会自动在其节点名后追加一个由父节点维护的自增长整型数字。

### 版本

- 每个ZNode，ZooKeeper都会为其维护一个叫做`Stat`的数据结构，记录了ZNode的三个数据版本
  - version：当前ZNode的版本。
  - cversion：当前ZNode子节点的版本。
  - aversion：ZNode的ACL版本。

### 事件监听器（Watcher）

- ZooKeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper服务端会将事件通知到感兴趣的客户端上。

### 访问控制列表（ACL）

- ZooKeeper采用ACL（Access Control Lists）策略来进行权限控制，类似于UNIX文件系统的权限控制，定义了如下5种权限
  - CREATE：创建子节点的权限。
  - READ：读取节点数据和子节点列表的权限。
  - WRITE：更新节点数据的权限。
  - DELETE：删除子节点的权限。
  - ADMIN：设置节点ACL的权限。
- 其中，CREATE和DELETE这两种权限是针对子节点的权限控制。

## 安装ZooKeeper

### 使用安装包安装

从[官网](http://zookeeper.apache.org/)下载安装包`zookeeper-3.4.13.tar.gz`，解压到`/opt`目录下：

``` shell
$ tar -zxf zookeeper-3.4.13.tar.gz -C /opt
```

### 创建目录

创建`data`和`logs`目录用于存储数据和日志：

``` shell
$ mkdir /data
$ mkdir /datalog
```

### 单机模式

在`conf`目录下创建配置文件`zoo.cfg`，填写以下内容：

``` 
# 端口号
clientPort=2181
# 数据目录
dataDir=/data
# 日志目录
dataLogDir=/datalog
# 心跳间隔时间，单位毫秒
tickTime=2000
```

进入ZooKeeper的`bin`目录，分别可以进行启动、停止、重启和查看节点状态操作：

``` shell
$ ./zkServer.sh start
$ ./zkServer.sh stop
$ ./zkServer.sh restart
$ ./zkServer.sh status
```

这样就完成了一个单机模式ZooKeeper的安装，但是单机模式下的ZooKeeper一旦发生故障，整个服务就会失效，并不具备实战意义，只适用于开发测试场景，生产上还是推荐ZooKeeper集群。

### 集群模式

搭建一个ZooKeeper集群，操作上也很简单，与单机模式基本类似，但至少需要3台服务器。

假设我们有3台服务器，IP地址分别是`172.17.0.2`，`172.17.0.3`，`172.17.0.4`，分别安装好ZooKeeper后，三个ZooKeeper的`conf/zoo.cfg`内容如下：

```
# 端口号
clientPort=2181
# 数据目录
dataDir=/data
# 日志目录
dataLogDir=/datalog
# 心跳间隔时间，单位毫秒
tickTime=2000
# 跟随者与领导者进行连接并同步的最大心跳数，在这时间内如果半数以上跟随者未能完成同步，则重新选举领导者
initLimit=5
# 跟随者与领导者进行同步的最大心跳数，在这时间内如果跟随者未完成同步，则将会被集群丢弃
syncLimit=2
# 节点配置，格式为 server.{id}={ip}:{port1}:{port2}
# port1用于领导者与跟随者的通信端口，port2用于参与竞选领导者的通信端口
server.1=172.17.0.2:2888:3888
server.2=172.17.0.3:2888:3888
server.3=172.17.0.4:2888:3888
```

分别修改三个ZooKeeper的`data/myid`文件，并对应`zoo.cfg`中配置的服务器id。

修改`172.17.0.2`的`data/myid`文件：

``` shell
$ echo '1' > data/myid
```

修改`172.17.0.3`的`data/myid`文件：

```shell
$ echo '2' > data/myid
```

修改`172.17.0.4`的`data/myid`文件：

```shell
$ echo '3' > data/myid
```

分别启动三个ZooKeeper节点，然后执行命令验证一下状态：

``` shell
$ ./zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.4.13/conf/zoo.cfg
Mode: follower
```

状态显示当前节点角色是`follower（跟随者）`，当然也可能是`leader（领导者）`，说明ZooKeeper集群已经搭建成功并且运行正常。

## ZooKeeper的基本用法

### 客户端连接

连接ZooKeeper可以使用命令行客户端，也可以使用Java客户端等方式，这里以ZooKeeper自带的命令行客户端为例。

进入ZooKeeper的	`bin`目录，执行：

``` shell
$ ./zkCli.sh
```

看到如下信息，表示已经成功连接上本地的ZooKeeper服务器：

```
[zk: localhost:2181(CONNECTED) 0]
```

如果想要连接指定的ZooKeeper服务器，则带上IP地址和端口：

``` shell
$ ./zkCli.sh -server 172.17.0.2:2181
```

### 读取

使用`ls`命令，可以列出指定节点下的所有子节点：

``` shell
[zk: localhost:2181(CONNECTED) 0] ls /

[zookeeper]
```

可以看到，默认有一个叫`/zookeeper`的保留节点。

还可以使用`get`命令，可以获取指定节点的数据内容和属性信息：

``` shell
[zk: localhost:2181(CONNECTED) 0] get /zookeeper

cZxid = 0x0
ctime = Thu Jan 01 00:00:00 GMT 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 GMT 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

### 创建

使用`create`命令，创建一个节点名为`/test`，数据内容为`123`：

```shell
[zk: localhost:2181(CONNECTED) 0] create /test 123
```

查看一下刚创建的节点数据和信息：

``` shell
[zk: localhost:2181(CONNECTED) 0] get /test

123
cZxid = 0x10000000e
ctime = Sun Nov 11 13:39:05 GMT 2018
mZxid = 0x10000000e
mtime = Sun Nov 11 13:39:05 GMT 2018
pZxid = 0x10000000e
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

默认创建的是持久节点，并且没有权限控制。如果希望创建的是临时节点，可以添加`-e`参数，如果希望创建的是顺序节点，可以添加`-s`参数，例如：

```shell
[zk: localhost:2181(CONNECTED) 0] create -e /test 123
```

### 更新

使用`set`命令，可以更新指定节点的数据内容：

``` shell
[zk: localhost:2181(CONNECTED) 0] set /test 456

cZxid = 0x10000000e
ctime = Sun Nov 11 13:39:05 GMT 2018
mZxid = 0x10000000f
mtime = Sun Nov 11 13:41:35 GMT 2018
pZxid = 0x10000000e
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

可以看到`dataVersion`从0变成了1。

### 删除

使用`delete`命令，可以删除指定节点：

``` shell
[zk: localhost:2181(CONNECTED) 0] delete /test
```

不过，要成功删除一个节点，该节点必须没有子节点，否则会收到一条出错信息：

``` shell
[zk: localhost:2181(CONNECTED) 0] delete /test
Node not empty: /test
```

## 参考文档

[《从Paxos到Zookeeper》](https://book.douban.com/subject/26292004/)

[https://zookeeper.apache.org/doc/current/index.html](https://zookeeper.apache.org/doc/current/index.html)

[https://www.cnblogs.com/lsdb/p/7297731.html](https://www.cnblogs.com/lsdb/p/7297731.html)

[https://mp.weixin.qq.com/s?__biz=MzIxNjA5MTM2MA==&mid=2652434980&idx=1&sn=6c43d646db29b0e2be12f7ca9d22bb2a](https://mp.weixin.qq.com/s?__biz=MzIxNjA5MTM2MA==&mid=2652434980&idx=1&sn=6c43d646db29b0e2be12f7ca9d22bb2a)