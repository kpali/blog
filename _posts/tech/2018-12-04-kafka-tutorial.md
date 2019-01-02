---
layout: post
title: Kafka简明教程
description: "关于Kafka Kafka是一个高吞吐量、分布式的发布-订阅消息系统，也是一个分布式流式处理平台，它最初由LinkedIn公司开发，后来成为Apache项目的一部分。Kafka核心模块使用Scala语言开发，支持多语言（如Java、C/C++、Python、Go、Erlang、Node.js等）客户端，它以可水平扩展和具有高吞吐量等特性而被广泛使用。"
category: 技术
tags: Kafka
---

## 关于Kafka

Kafka是一个高吞吐量、分布式的发布-订阅消息系统，也是一个分布式流式处理平台，它最初由LinkedIn公司开发，后来成为Apache项目的一部分。Kafka核心模块使用Scala语言开发，支持多语言（如Java、C/C++、Python、Go、Erlang、Node.js等）客户端，它以可水平扩展和具有高吞吐量等特性而被广泛使用。

## Kafka的基本概念

### 消息（Message或Record）

消息是Kafka通信的基本单位，由一个固定长度的消息头和一个可变长度的消息体构成。

### 主题（Topic）

Kafka将一组消息抽象归纳为一个主题，也就是说，一个主题就是对消息的一个分类。

### 分区（Partition）

Kafka将一个主题分为若干个分区，一个分区就是一个提交日志。Kafka通过分区来实现数据冗余和伸缩性，分区可以分布在不同的服务器上，也就是说一个主题可以横跨多个服务器，以此来提升性能和可用性。

单个分区内的消息是一个有序队列，但Kafka只能保证一个分区之内消息的有序性，并不能保证跨分区消息的有序性。

### 副本（Replica）

一个分区包含多个副本，需要保证一个分区的多个副本之间数据的一致性，因此Kafka会选择一个副本作为Leader副本，该分区的其他副本则为Follower副本，只有Leader副本可以处理分区的所有读/写请求，Follower副本定期从Leader副本同步数据。

### 代理（Broker）

一台Kafka服务器就是一个代理，一个Kafka集群由多个代理组成。

### 偏移量（Offset）

偏移量指的是消息在分区中的位置，由消费者保存和控制，消费者通过检查消息的偏移量来区分已经读取过的消息，也可以改变偏移量处理过去的消息等等。

### 生产者（Producer）

生产者创建消息发布到指定的主题，消息会发送到Kafka代理。

默认情况下消息均衡地分布到主题的分区上，但生产者也能基于一些的算法决定消息的所属分区。

### 消费者（Consumer）

消费者向指定主题获取消息。

### 消费者组（ConsumerGroup）

一个消费者组可以包含多个消费者，我们可以为每个消费者指定一个消费者组，如果不指定，则该消费者属于默认消费者组。

一个主题下的某个分区只能被消费者组下的某个消费者消费，当然该分区还可以分配给其他消费者组。因此如果同一个消费者组的消费者数量超过订阅主题的分区数量，则会有消费者被闲置。

消费者组是Kafka用来实现一个主题消息进行广播和单播的手段，实现消息广播只需指定各消费者均属于不同的消费者组，消息单播则只需让各消费者属于同一个消费者组。

### ZooKeeper

ZooKeeper是一个分布式应用程序协调服务，并不是Kafka的一部分，Kafka利用ZooKeeper保存相应元数据信息，Kafka元数据信息包括如代理节点信息、Kafka集群信息、旧版消费者信息及消费偏移量信息、主题信息、分区状态信息、分区副本分配方案信息、动态配置信息等。

## Kafka的特性

### 消息持久化

消息持久化是Kafka的一个重要特性，Kafka把收到的消息都写入到硬盘中，可以保证数据不会丢失，并且允许消费者非实时地读取消息。消费者如果因为一些原因离线了一小段时间，可以从上次中断的地方继续处理消息。

Kafka提供了两种策略来删除数据，一种是基于时间，保留多长时间（比如7天），另一种是基于分区文件大小，保留多大数据量（比如1GB），每个主题可以设置单独的保留规则。

### 高吞吐量

Kafka实现高吞吐量的原因主要有：

1. 顺序I/O

   Kafka使用了顺序I/O，并极力避免随机磁盘访问。前者的写入速度比后者快了一个数量级。Kafka所采用的提交日志就是以追加的方式写入分区的，就是说单个分区的写入是可以保证顺序的，没有删除和更新操作，因此避免了随机写入。另外，从分区读取数据的时候也是按顺序读取的，避免了随机读取。

2. 内存映射文件（Memory Mapped Files）

   内存映射文件将磁盘上的文件内容与内存映射起来，我们往内存里写入数据，操作系统会在稍后把数据冲刷到磁盘上。所以，在写入数据时几乎就是写入内存的速度。

3. 零拷贝（Zero-Copy）

   Kafka在数据写入及数据同步采用了零拷贝技术，避免了内核缓冲区与用户缓冲区之间数据的拷贝，操作效率极高。

4. 应用层面的优化

   除了利用底层的技术外，Kafka还在应用程序层面提供了一些手段来提升性能，例如压缩、批量发送和分区。最明显的就是使用批次，在向Kafka写入数据时，可以启用批次写入，这样可以避免在网络上频繁传输单个消息带来的延迟和带宽开销。假设网络带宽为10MB/S，一次性传输10MB的消息比传输1KB的消息10000万次显然要快得多。

### 伸缩性

为了能够轻松处理大量数据，Kafka从一开始就被设计成一个具有灵活伸缩性的系统。Kafka依赖ZooKeeper来对集群进行协调管理，这样使得Kafka更加容易进行水平扩展，生产者、消费者和代理都为分布式，可配置多个，集群扩展时无需将整个集群停机。

## Kafka的应用场景

1. 消息系统

   Kafka作为一款优秀的消息系统，具有高吞吐量、内置的分区、备份冗余分布式等特点，为大规模消息处理提供了一种很好的解决方案。

2. 应用监控

   利用Kafka采集应用程序和服务器健康相关的指标，如CPU占用率、IO、内存、连接数、TPS、QPS等，然后将指标信息进行处理，从而构建一个具有监控仪表盘、曲线图等可视化监控系统。例如，很多公司采用Kafka与ELK（ElasticSearch、Logstash和Kibana）整合构建应用服务监控系统。

3. 网站用户行为追踪

   为了更好地了解用户行为、操作习惯，改善用户体验，进而对产品升级改进，讲用户操作轨迹、内容等信息发送到Kafka集群上，通过Hadoop、Spark或Strom等进行数据分析处理，生成相应的统计报告，为推荐系统推荐对象建模提供数据源，进而为每个用户进行个性化推荐。

4. 流处理

   需要将己收集的流数据提供给其他流式计算框架进行处理，用Kafka收集流数据是一个不错的选择，而且当前版本的Kafka提供了Kafka Streams支持对流数据的处理 

5. 持久性日志

   Kafka可以为外部系统提供一种持久性日志的分布式系统。日志可以在多个节点间进行备份 ，Kafka为故障节点数据恢复提供了 一种重新同步的机制。同时，Kafka很方便与HDFS和Flume进行整合，这样就方便将Kafka采集的数据持久化到其他外部系统 。

## 安装Kafka

假设我们有3台服务器，分别是`192.168.232.154`、`192.168.232.155`、`192.168.232.156`，3台操作系统均为CentOS 7，用来组成Kafka集群，以下安装步骤以`192.168.232.154`为例，但需要在3台服务器上分别操作。

### 安装Java

安装Java1.8，并配置好`JAVA_HOME`、`PATH`等环境变量。

### 配置/etc/hosts

编辑`/etc/hosts`文件，加入3台服务器的IP和主机名映射：

```
192.168.232.154 kafka-a
192.168.232.155 kafka-b
192.168.232.156 kafka-c
```

### 安装ZooKeeper

由于Kafka需要依赖ZooKeeper，所以还需要安装ZooKeeper，这里方便起见直接利用3台准备安装Kafka的服务器来组成ZooKeeper集群（或者直接使用Kafka自带的ZooKeeper也行）。

关于ZooKeeper的集群安装可以参考[《ZooKeeper简明教程》](http://kpali.me/2018/11/07/zookeeper-tutorial.html)。

### 安装Kafka

从[官网](http://kafka.apache.org/)下载安装包`kafka_2.12-2.1.0.tgz`，前一个`2.12`表示Scala版本，后一个`2.1.0`表示Kafka版本，解压到`/opt`目录下：

``` shell
$ tar -zxf kafka_2.12-2.1.0.tgz -C /opt
```

创建Kafka日志存储目录:

``` shell
$ mkdir /kafka-logs
```

修改`server.properties`配置文件，找到并修改`broker.id`、`log.dirs`、`listeners`和`zookeeper.connect`，其他配置保持不变：

```
# 代理ID
broker.id=1
# 日志存储路径
log.dirs=/kafka-logs
# 监听绑定的网络接口和端口，提供外部访问，可以使用IP地址或主机名等，这里使用IP地址
listeners=PLAINTEXT://192.168.232.154:9092
# 要连接的ZooKeeper，多个ZooKeeper之间用逗号分隔
zookeeper.connect=192.168.232.154:2181,192.168.232.155:2181,192.168.232.156:2181
```

其中`broker.id`需要保证唯一，因此三个Kafka节点则分别设置为1，2，3。

最后进入Kafka的`bin`目录，然后分别启动3个Kafka：

``` shell
$ ./kafka-server-start.sh -daemon ../config/server.properties
```

### 验证Kafka启动状态

进入ZooKeeper的`bin`目录下，通过ZooKeeper客户端登陆到ZooKeeper：

``` shell
$ ./zkCli.sh
```

查看目录结构：

``` shell
[zk: localhost:2181(CONNECTED) 0] ls /

[cluster, controller_epoch, controller, brokers, zookeeper, admin, isr_change_notification, consumers, log_dir_event_notification, latest_producer_id_block, config]
```

查看当前已启动的Kafka代理节点：

``` shell
[zk: localhost:2181(CONNECTED) 1] ls /brokers/ids

[1, 2, 3]
```

结果显示当前有3个Kafka代理节点，分别是1，2，3。

## 使用Kafka

管理和使用Kafka有多种方式，这里主要介绍2种，一种是使用Kafka自带的脚本，另一种是使用Java API编程。

### 使用Kafka脚本

Kafka的`bin`目录下有若干个脚本，可以用于执行一些操作。

首先创建一个分区数量为4，副本数量为3的主题，主题名称为`my-topic`：

``` shell
$ ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 4 --topic my-topic

Created topic "my-topic".
```

查看一下这个主题在所有Kafka代理集群中的情况：

``` shell
$ ./kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-topic

Topic:my-topic  PartitionCount:4        ReplicationFactor:3     Configs:
        Topic: my-topic Partition: 0    Leader: 1       Replicas: 1,3,2 Isr: 1,3,2
        Topic: my-topic Partition: 1    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
        Topic: my-topic Partition: 2    Leader: 3       Replicas: 3,2,1 Isr: 3,2,1
        Topic: my-topic Partition: 3    Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
```

第一行是所有分区的摘要，显示该主题有4个分区，3个副本。之后是每个分区的信息，以第二行为例：

- 信息显示编号为`0`的分区的`Leader`是代理节点1。

- `Replicas`表示该分区副本所在的节点列表，无论这些节点是否存活。信息显示分区`0`的3个副本分布在代理节点1，3，2上。
- `Isr`表示In-Sync Replicas（同步副本），表示存活并且与`Leader`正常同步的副本所在节点。信息显示副本所在节点1，3，2均存活且正常同步。

试着发布一些消息到该主题：

``` shell
$ ./kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
>my test message 1
>my test message 2
>^C
```

一共发布了2条消息，并按`CTRL+C`退出。然后消费刚刚发布的消息：

``` shell
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-topic
my test message 1
my test message 2
^C
```

获取后按`CTRL+C`退出，其中`--from-beginning`选项表示从消息最开始读取。

### 使用Java API客户端

添加Maven依赖：

```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.1.0</version>
</dependency>
```

一个简单的Demo：

``` java
import org.apache.kafka.clients.CommonClientConfigs;
import org.apache.kafka.clients.admin.AdminClient;
import org.apache.kafka.clients.admin.NewTopic;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;

import java.time.Duration;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.Properties;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class KafkaDemoApplication {

    public static void main(String[] args) {
        
        Properties props = new Properties();
        props.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG,
                "192.168.232.154:9092,192.168.232.155:9092,192.168.232.156:9092");
        AdminClient adminClient = AdminClient.create(props)；
        // 设置主题名称，并配置4个分区，3个副本
        String topicName = "demoTopic";
        NewTopic newTopic = new NewTopic(topicName, 4, (short)3);
        Collection<NewTopic> newTopics = new ArrayList<>();
        newTopics.add(newTopic);
        // 创建主题
        adminClient.createTopics(newTopics);
        adminClient.close();

        int corePoolSize = Runtime.getRuntime().availableProcessors() * 2;
        ThreadPoolExecutor threadsPool;
        threadsPool = new ThreadPoolExecutor(corePoolSize, corePoolSize, 60, TimeUnit.SECONDS, new LinkedBlockingDeque<>());
        
        threadsPool.execute(() -> {
            // 创建生产者
            Properties producerProps = new Properties();
            producerProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.232.154:9092,192.168.232.155:9092,192.168.232.156:9092");
            producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
            producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
            Producer<String, String> producer = new KafkaProducer<String, String>(producerProps);
            try {
                while (true) {
                    // 发送消息
                    String str = String.valueOf((int)(Math.random() * 10000));
                    ProducerRecord<String, String> record = new ProducerRecord<>(topicName, str, str);
                    producer.send(record);
                    System.out.println("生产者：" + str);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            finally {
                producer.close();
            }
        });
        
        threadsPool.execute(() -> {
            // 创建消费者
            Properties consumerProps = new Properties();
            consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.232.154:9092,192.168.232.155:9092,192.168.232.156:9092");
            consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "MyConsumers");
            consumerProps.put(ConsumerConfig.CLIENT_ID_CONFIG, "MyConsumer");
            consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
            consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
            Consumer<String, String> consumer= new KafkaConsumer<String, String>(consumerProps);
            // 订阅主题
            consumer.subscribe(Collections.singletonList(topicName));
            try {
                while (true) {
                    // 消息轮询
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
                    for (ConsumerRecord<String , String> record : records) {
                        System.out.println(String.format("消费者：topic = %s, partition = %s, offset = %d, key = %s, value = %s",
                                record.topic(), record.partition(), record.offset(), record.key(), record.value()));
                    }
                }
            } catch (RuntimeException e) {
                e.printStackTrace();
            }
            finally {
                consumer.close();
            }
        });
        
    }
}
```

## 参考文档

[http://kafka.apache.org/documentation.html](http://kafka.apache.org/documentation.html)

[《Kafka权威指南》](https://book.douban.com/subject/27665114/)

[《Kafka入门与实践》](https://book.douban.com/subject/27174117/)

[《Kafka源码解析与实战》](https://book.douban.com/subject/30128444/)

[https://www.cnblogs.com/huxi2b/p/6223228.html](https://www.cnblogs.com/huxi2b/p/6223228.html)

[https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945468&idx=1&sn=b622788361b384e152080b60e5ea69a7](https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945468&idx=1&sn=b622788361b384e152080b60e5ea69a7)

[https://mp.weixin.qq.com/s?__biz=MzI4Mzc0NTc4MQ==&mid=2247483831&idx=1&sn=5c27923ff0c6f720ac04d4b892fa51e0&chksm=eb874d2ddcf0c43ba4fd931f91409b6a61d0e3b5cebe5f29806a3db655f3f86b94866b8ebdd1](https://mp.weixin.qq.com/s?__biz=MzI4Mzc0NTc4MQ==&mid=2247483831&idx=1&sn=5c27923ff0c6f720ac04d4b892fa51e0&chksm=eb874d2ddcf0c43ba4fd931f91409b6a61d0e3b5cebe5f29806a3db655f3f86b94866b8ebdd1)