---
layout: post
title: Redis简明教程
description: "关于Redis Redis是一个开源的高性能key-value数据库，Redis与其他key-value缓存产品有以下三个特点： Redis除了能存储普通的字符串键之外，还可以存储其他4种数据结构：列表、集合、散列、有序集合。 Redis支持数据的持久化，可以将内存中的数据保存在磁盘中。 Redis支持主从复制，可以实时对数据进行备份，同时支持读写分离。"
category: 技术
tags: Redis
---

## 关于Redis

Redis是一个开源的高性能key-value内存数据库，主要有以下三个特点：

- Redis可以存储5种数据结构：STRING（字符串）、LIST（列表）、HASH（哈希）、SET（集合）、ZSET（有序集合）。
- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中。
- Redis支持主从复制，可以实时对数据进行备份，同时支持读写分离。

## 安装Redis

### CentOS7.5下安装Redis

下载redis并解压，这里最新稳定版为5.0.0：

``` shell
# wget http://download.redis.io/releases/redis-5.0.0.tar.gz
# tar xzf redis-5.0.0.tar.gz
# cd redis-5.0.0
```

安装gcc：

``` shell
# yum install gcc
```

编译安装：

``` shell
# make MALLOC=libc
# cd src && make install
```

测试是否安装成功：

``` shell
# ./redis-server
```

出现Redis启动信息表示安装成功，`Ctrl + C`退出测试。

设置Redis开机自启动，在`/etc`目录下新建`redis`目录：

``` shell
# mkdir -p /etc/redis
```

将`redis.conf`文件复制一份到`/etc/redis`目录下，并命名为`6379.conf`：

``` shell
# cp redis.conf /etc/redis/6379.conf
```

修改`6379.conf`，找到`daemonize no`一行，设置Redis为后台启动：

```
daemonize yes
```

将Redis的启动脚本复制一份到`/etc/init.d`目录下：

``` shell
# cp utils/redis_init_script /etc/init.d/redisd
```

设置Redis开机自启动：

``` shell
# cd /etc/init.d
# chkconfig redisd on
```

启动Redis：

``` shell
# service redisd start
```

### 使用Docker安装Redis

如果会使用Docker的话，可以更方便快速地安装和体验Redis，只要下载一个Redis的镜像即可：

``` shell
$ docker pull redis
```

启动容器，映射`6379`端口：

``` shell
$ docker run -d --name redis -p 6379:6379 redis
```

## Redis数据结构

Redis提供的5种基本结构：

| 结构类型         | 结构存储的值                                                 | 结构的读写能力                                               |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STRING           | 可以是字符串、整数或者浮点数                                 | 对整个字符串或者字符串的其中一部分执行操作；对整数和浮点数执行自增（increment）或者自减（decrement）操作 |
| LIST             | 一个链表，链表上的每个节点都包含了一个字符串                 | 从链表的两端推入或者弹出元素；根据偏移量对链表进行修剪（trim）；读取单个或者多个元素；根据值查找或者移除元素 |
| HASH             | 包含键值对的无序哈希表                                       | 添加、获取、移除单个键值对；获取所有键值对                   |
| SET              | 包含字符串的无序收集器（unordered collection），并且被包含的每个字符串都是独一无二、各不相同的 | 添加、获取、移除单个元素；检查一个元素是否存在于集合中；计算交集、并集、差集；从集合里面随机获取元素 |
| ZSET（有序集合） | 字符串成员（member）与浮点数分值（score）之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、删除单个元素；根据分值范围（range）或者成员来获取元素 |

### 字符串（String）

字符串的基本命令：

- GET：获取存储在给定键中的值
- SET：设置存储在给定键中的值
- DEL：删除存储在给定键中的值（这个命令可以用于所有类型）

使用`redis-cli`与Redis进行交互：

``` shell
$ redis-cli
127.0.0.1:6379> 
```

将键hello的值设置为world：

``` shell
127.0.0.1:6379> set hello world
OK
```

获取存储在键hello中的值：

``` shell
127.0.0.1:6379> get hello
"world"
```

删除键值对：

``` shell
127.0.0.1:6379> del hello
(integer) 1
```

再次获取存储在键hello中的值：

``` shell
127.0.0.1:6379> get hello
(nil)
```

### 列表（List）

列表的基本命令：

- RPUSH：将给定值推入列表的右端
- LRANGE：获取列表在给定范围上的所有值
- LINDEX：获取列表在给定位置上的单个元素
- LPOP：从列表的左端弹出一个值，并返回被弹出的值

假设列表key为`my-list`，向列表分别推入值`a`、`b`和`c`：

``` shell
127.0.0.1:6379> rpush my-list a
(integer) 1
127.0.0.1:6379> rpush my-list b
(integer) 2
127.0.0.1:6379> rpush my-list c
(integer) 3
```

获取列表所有值（`0 -1` 0为起始索引，-1为结束索引，表示取出所有值）：

``` shell
127.0.0.1:6379> lrange my-list 0 -1
1) "a"
2) "b"
3) "c"
```

获取列表中索引为1的值：

``` shell
127.0.0.1:6379> lindex my-list 1
"b"
```

从列表中弹出一个元素，弹出的元素将不再存在于列表：

``` shell
127.0.0.1:6379> lpop my-list
"a"
127.0.0.1:6379> lrange my-list 0 -1
1) "b"
2) "c"
```

### 哈希（Hash）

哈希的基本命令：

- HSET：在哈希表里面关联起给定的键值对
- HGET：获取存储在哈希表中指定键的值
- HGETALL：获取哈希表包含的所有键值对
- HDEL：如果给定键存在于哈希表里面，那么移除这个键

假设哈希表key为`my-hash`，分别向哈希表添加键值对：

```shell
127.0.0.1:6379> hset my-hash a 1
(integer) 1
127.0.0.1:6379> hset my-hash b 2
(integer) 1
127.0.0.1:6379> hset my-hash c 3
(integer) 1
```

获取键为`a`的值：

```shell
127.0.0.1:6379> hget my-hash a
"1"
```

获取哈希表所有键值对：

```shell
127.0.0.1:6379> hgetall my-hash
1) "a"
2) "1"
3) "b"
4) "2"
5) "c"
6) "3"
```

根据键删除键值对：

```shell
127.0.0.1:6379> hdel my-hash a
(integer) 1
127.0.0.1:6379> hgetall my-hash
1) "b"
2) "2"
3) "c"
4) "3"
```

### 集合（Set）

集合的基本命令：

- SADD：将给定元素添加到集合
- SMEMBERS：返回集合包含的所有元素
- SISMEMBER：检查给定元素是否存在于集合中
- SREM：如果给定的元素存在于集合中，那么移除这个元素

集合和列表都可以存储多个字符串，区别在于：

- 列表可以存储多个相同的字符串，集合则通过使用散列表来保证自己存储的每个字符串都是不相同的
- 集合是以无序的方式存储元素

假设集合key为`my-set`，向集合添加值：

``` shell
127.0.0.1:6379> sadd my-set a
(integer) 1
127.0.0.1:6379> sadd my-set b
(integer) 2
127.0.0.1:6379> sadd my-set c
(integer) 3
```

重复添加值`a`：

``` shell
127.0.0.1:6379> sadd my-set a
(integer) 0
```

返回0表示这个元素已经存在于集合中，获取所有元素可以看到只有一个`a`：

``` shell
127.0.0.1:6379> smembers my-set
1) "a"
2) "b"
3) "c"
```

检查元素是否存在于集合中：

``` shell
127.0.0.1:6379> sismember my-set d
(integer) 0
127.0.0.1:6379> sismember my-set a
(integer) 1
```

移除集合中的元素：

``` shell
127.0.0.1:6379> srem my-set a
(integer) 1
127.0.0.1:6379> srem my-set a
(integer) 0
127.0.0.1:6379> smembers my-set
1) "b"
2) "c"
```

### 有序集合（Zset、Sorted set）

有序集合的基本命令：

- ZADD：将一个带有给定分值的成员添加到有序集合里面
- ZRANGE：根据元素在有序排列中所处的位置，从有序集合里面获取多个元素
- ZRANGEBYSCORE：获取有序集合在给定分值范围内的所有元素
- ZREM：如果给定成员存在于有序集合，那么移除这个成员

有序集合与集合区别在于：

- 有序集合是键值对存储，和哈希类似。
- 有序集合的值称为分值（score），分值是double类型，通过分值来进行排序，分值可以重复。

假设有序集合key为`my-zset`，向有序集合添加值，其中，1、2、3分别是a、b、c的分值：

``` shell
127.0.0.1:6379> zadd my-zset 1 a
(integer) 1
127.0.0.1:6379> zadd my-zset 2 b
(integer) 1
127.0.0.1:6379> zadd my-zset 3 c
(integer) 1
```

获取有序集合的所有元素：

``` shell
127.0.0.1:6379> zrange my-zset 0 -1
1) "a"
2) "b"
3) "c"
```

获取有序集合的所有元素，附上分值：

``` shell
127.0.0.1:6379> zrange my-zset 0 -1 withscores
1) "a"
2) "1"
3) "b"
4) "2"
5) "c"
6) "3"
```

根据分值范围获取有序集合的元素，分值范围为0-10：

``` shell
127.0.0.1:6379> zrangebyscore my-zset 0 10
1) "a"
2) "b"
3) "c"
```

同样可以附上分值：

``` shell
127.0.0.1:6379> zrangebyscore my-zset 0 10 withscores
1) "a"
2) "1"
3) "b"
4) "2"
5) "c"
6) "3"
```

移除指定成员：

``` shell
127.0.0.1:6379> zrem my-zset a
(integer) 1
127.0.0.1:6379> zrange my-zset 0 -1
1) "b"
2) "c"
```

### HyperLogLog

HyperLogLog的基本命令：

- PFADD：添加指定元素到HyperLogLog中
- PFCOUNT：返回给定HyperLogLog的基数估算值
- PFMERGE：将多个HyperLogLog合并为一个HyperLogLog

Redis在2.8.9版本添加了HyperLogLog结构，用于做基数统计，即估算不重复元素的数量。

**优点**是只需要花费12KB内存，就可以计算接近2的64次方个不同元素的基数。

**缺点**是由于是估算，因此存在一定误差，而且不会存储元素本身，无法返回输入的各个元素。

HyperLogLog可以用在对精确性要求不是很高的场景，比如统计在线人数等等。

示例：

``` shell
127.0.0.1:6379> pfadd my-hll a
(integer) 1
127.0.0.1:6379> pfadd my-hll b
(integer) 1
127.0.0.1:6379> pfcount my-hll
(integer) 2
```

## 设置数据过期时间

Redis支持为数据设置过期时间（单位：秒），并在过期后自动销毁数据.

示例，添加数据，并设置保留5秒：

``` shell
127.0.0.1:6379> set my-key my-value
OK
127.0.0.1:6379> expire my-key 5
(integer) 1
```

5秒内获取：

``` shell
127.0.0.1:6379> get my-key
"my-value"
```

5秒后获取：

``` shell
127.0.0.1:6379> get my-key
(nil)
```

## 更多的命令

除了上面介绍的基本命令外，Redis还提供了许多命令来支持不同场景下的需求，详情可以查阅官方文档：

[https://redis.io/commands](https://redis.io/commands)

## 发布订阅

发布订阅的基本命令：

- SUBSCRIBE：订阅给定的一个或者多个频道的信息
- PUBLISH：将信息发送到指定的频道
- UNSUBSCRIBE：退订给定的频道

Redis发布订阅通过建立一个频道（channel），发布者（pub）发送消息，订阅者（sub）接收消息，实现发布者与订阅者之间的消息通信。

示例：

订阅频道`my-channel` :

``` shell
127.0.0.1:6379> subscribe my-channel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "my-channel"
3) (integer) 1
```

然后重新打开一个客户端，向频道`my-channel`发布一条消息：

``` shell
127.0.0.1:6379> publish my-channel "hello"
(integer) 1
```

前面订阅者的客户端就会收到这条消息，如下：

``` shell
127.0.0.1:6379> subscribe my-channel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "my-channel"
3) (integer) 1
1) "message"
2) "my-channel"
3) "hello"
```

## 事务

事务的基本命令：

- MULTI：标记一个事务的开始
- EXEC：执行事务队列的所有命令
- DISCARD：取消事务，并放弃已经进入事务队列的所有命令
- WATCH：监视一个或多个key，如果在事务执行之前这些key被修改过，则事务将失败。
- UNWATCH：取消对所有key的监视

示例：

``` shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> sadd my-set a
QUEUED
127.0.0.1:6379> sadd my-set b
QUEUED
127.0.0.1:6379> smembers my-set
QUEUED
127.0.0.1:6379> exec
1) (integer) 1
2) (integer) 1
3) 1) "a"
   2) "b"
```

**注意：**

Redis的事务与关系型数据库事务有所不同，单条Redis命令是原子操作，但是Redis事务并不是原子操作，并且Redis事务没有回滚，只是批量执行命令而已。

因为Redis的设计目标是高性能，如果和关系型数据库一样，通过加锁来实现事务的原子操作，性能势必大打折扣，而通过`WATCH`命令，如果监视的key发生改变则事务执行失败，需要重新执行事务，这是乐观锁的做法，在实际使用中也能解决问题。

## 备份与恢复

使用`save`命令创建备份，会在Redis的安装目录下生成备份文件`dump.rdb`：

``` shell
127.0.0.1:6379> save
OK
```

Redis的安装目录可以通过以下命令获取到：

``` shell
127.0.0.1:6379> config get dir
1) "dir"
2) "/data"
```

但是`save`命令会阻止所有其他客户端，所以通常使用另一个命令`bgsave`：

``` shell
27.0.0.1:6379> bgsave
Background saving started
```

使用`bgsave`备份会在后台执行，并且Redis会进行fork，父进程继续提供数据库服务，子进程将数据库保存到磁盘上。要检查备份是否完成，可以使用`lastsave`命令：

``` shell
127.0.0.1:6379> lastsave
(integer) 1541486211
```

返回的是上次备份成功的UNIX时间戳。

## 性能测试

Redis提供了一个性能测试工具`redis-benchmark`用来检测性能：

模拟同时执行10000个请求：

``` shell
$ redis-benchmark -n 10000 -q

PING_INLINE: 13698.63 requests per second
PING_BULK: 12870.01 requests per second
SET: 13333.33 requests per second
GET: 15220.70 requests per second
INCR: 11286.68 requests per second
LPUSH: 11614.40 requests per second
RPUSH: 11415.53 requests per second
LPOP: 13037.81 requests per second
RPOP: 15649.45 requests per second
SADD: 13054.83 requests per second
HSET: 13550.14 requests per second
SPOP: 13333.33 requests per second
LPUSH (needed to benchmark LRANGE): 13531.80 requests per second
LRANGE_100 (first 100 elements): 10162.60 requests per second
LRANGE_300 (first 300 elements): 5672.15 requests per second
LRANGE_500 (first 450 elements): 3952.57 requests per second
LRANGE_600 (first 600 elements): 3271.18 requests per second
MSET (10 keys): 9891.20 requests per second
```

仅测试`GET`和`RPUSH`命令的性能：

``` shell
$ redis-benchmark -h 127.0.0.1 -p 6379 -t get,rpush -n 10000 -q

GET: 16666.67 requests per second
RPUSH: 15337.42 requests per second
```

``` shell
127.0.0.1:6379> save
OK
```

## 搭建主从复制



## 参考文档

[《Redis实战》](https://book.douban.com/subject/26612779/)

[http://www.runoob.com/redis/redis-tutorial.html](http://www.runoob.com/redis/redis-tutorial.html)

[https://www.cnblogs.com/zuidongfeng/p/8032505.html](https://www.cnblogs.com/zuidongfeng/p/8032505.html)

