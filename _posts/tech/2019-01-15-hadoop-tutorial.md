---
layout: post
titile: Hadoop简明教程
description: "关于Hadoop Apache Hadoop是一款支持数据密集型分布式应用程序并以Apache 2.0许可协议发布的开源软件框架。它支持在商品硬件构建的大型集群上运行的应用程序。Hadoop是根据谷歌公司发表的MapReduce和Google文件系统的论文自行实现而成。所有的Hadoop模块都有一个基本假设，即硬件故障是常见情况，应该由框架自动处理。"
category: 技术
tags: [ Hadoop, HDFS, YARN ]
---

## 关于Hadoop

Apache Hadoop是一款支持数据密集型分布式应用程序并以Apache 2.0许可协议发布的开源软件框架。它支持在商品硬件构建的大型集群上运行的应用程序。Hadoop是根据谷歌公司发表的MapReduce和Google文件系统的论文自行实现而成。所有的Hadoop模块都有一个基本假设，即硬件故障是常见情况，应该由框架自动处理。

Hadoop框架透明地为应用提供可靠性和数据移动。它实现了名为MapReduce的编程范式：应用程序被分割成许多小部分，而每个部分都能在集群中的任意节点上运行或重新运行。此外，Hadoop还提供了分布式文件系统，用以存储所有计算节点的数据，这为整个集群带来了非常高的带宽。MapReduce和分布式文件系统的设计，使得整个框架能够自动处理节点故障。它使应用程序与成千上万的独立计算的计算机和PB级的数据连接起来。现在普遍认为整个Apache Hadoop“平台”包括Hadoop内核、MapReduce、Hadoop分布式文件系统（HDFS）以及一些相关项目，有Apache Hive和Apache HBase等等。

## Hadoop的基本概念

### MapReduce

MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算。概念"Map（映射）"和"Reduce（归约）"，是它们的主要思想，都是从函数式编程语言里借来的，还有从矢量编程语言里借来的特性。它极大地方便了编程人员在不会分布式并行编程的情况下，将自己的程序运行在分布式系统上。 当前的软件实现是指定一个Map（映射）函数，用来把一组键值对映射成一组新的键值对，指定并发的Reduce（归约）函数，用来保证所有映射的键值对中的每一个共享相同的键组。

简单来说，一个映射函数就是对一些独立元素组成的概念上的列表（例如，一个测试成绩的列表）的每一个元素进行指定的操作（比如，有人发现所有学生的成绩都被高估了一分，他可以定义一个“减一”的映射函数，用来修正这个错误。）。事实上，每个元素都是被独立操作的，而原始列表没有被更改，因为这里创建了一个新的列表来保存新的答案。这就是说，Map操作是可以高度并行的，这对高性能要求的应用以及并行计算领域的需求非常有用。

而归纳操作指的是对一个列表的元素进行适当的合并（继续看前面的例子，如果有人想知道班级的平均分该怎么做？他可以定义一个归纳函数，通过让列表中的奇数（odd）或偶数（even）元素跟自己的相邻的元素相加的方式把列表减半，如此递归运算直到列表只剩下一个元素，然后用这个元素除以人数，就得到了平均分）。虽然他不如映射函数那么并行，但是因为归纳总是有一个简单的答案，大规模的运算相对独立，所以归纳函数在高度并行环境下也很有用。 

### HDFS

HDFS是一个分布式文件系统。因为HDFS具有高容错性的特点，所以它可以设计部署在低廉的硬件上。她可以通过提供高吞吐率来访问应用程序的程序，适合那些有着超大数据集的应用程序。HDFS放宽了对可移植操作系统接口（POSIX）的要求，这样可以实现以流的形式访问文件系统中的数据。HDFS原本是开源的Apache项目Nutch的基础结构，最后它却成为了Hadoop基础架构之一。

### YARN

Apache YARN（Yet Another Resource Negotiator）是Hadoop的集群资源管理系统。YARN被引入Hadoop 2，最初是为了改善MapReduce的实现，但它具有足够的通用性，同样可以支持其他的分布式计算模式。

YARN提供请求和使用集群资源的API，但这些API很少直接用于用户代码。相反，用户代码中用的是分布式计算框架提供的更高层API，这些API建立在YARN之上且向用户隐藏了资源管理细节。一些分布式计算框架（MapReduce，Spark等等）作为YARN应用运行在集群计算层（YARN）和集群存储层（HDFS和HBase）上。还有一层应用如Pig、Hive和Crunch都是运行在MapReduce，Spark或Tez之上的处理框架，它们不和YARN直接打交道。

### Hive

Hive最早是由Facebook设计的，是一个建立在Hadoop基础之上的数据仓库，它提供了一些用于对Hadoop文件中的数据集进行数据整理、特殊查询和分析存储的工具。Hive提供的是一种结构化数据的机制，她支持类似于传统RDBMS中的SQL语言的查询语言，来帮助那些熟悉SQL的用户查询Hadoop中的数据，该查询语言称为Hive QL。与此同时，传统的MapReduce编程人员也可以在Mapper或Reducer中通过Hive QL查询数据。Hive编译器会把Hive QL编译成一组MapReduce任务，从而方便MapReduce编程人员进行Hadoop系统开发。

### HBase

HBase是一个分布式的、面向列的开源数据库，该技术来源于Google论文《Bigtable：一个结构化数据的分布式存储系统》。如果Bigtable利用了Google文件系统（Google File System）提供的分布式数据存储方式一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase不同于一般的关系数据库，原因有两个：其一，HBase是一个适合于非结构化数据存储的数据库；其二，HBase是基于列而不是基于行的模式。HBase和Bigtable使用相同的数据模型。用户将数据存储在一个表里，一个数据行拥有一个可选择的键和任意数量的列。由于HBase表是疏松的，用户可以为行定义各种不同的列。HBase主要用于需要随机访问、实时读写的大数据。

## 安装Hadoop

假设我们有3台服务器，分别是`hadoop-a`、`hadoop-b`、`hadoop-c`，3台操作系统均为CentOS 7，用来组成Hadoop集群，Hadoop集群规划：

**HDFS：**

hadoop-a：NameNode、SecondaryNameNode

hadoop-b：DataNode

hadoop-c：DataNode

**YARN：**

hadoop-a：ResouceManager

hadoop-b：NodeManager

hadoop-c：NodeManager

### 安装ssh和rsync

检查和安装ssh和rsync，ssh以后台服务运行，以便Hadoop脚本管理远程Hadoop守护进程。

### 安装Java

安装JAVA 8，这里安装的是OpenJDK 1.8，并配置好`JAVA_HOME`、`PATH`等环境变量。

### 配置/etc/hosts

编辑/etc/hosts文件，加入3台服务器的IP和主机名映射：

    192.168.232.161 hadoop-a
    192.168.232.162 hadoop-b
    192.168.232.163 hadoop-c

### 配置SSH免密码登录

生成密钥：

``` shell
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa
```

将公钥追加到本机的`authorized_keys`中：

``` shell
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

然后将另外两台服务器生成的`id_rsa.pub`内容也追加到`authorized_keys`中，最终如下：

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDUHjkpc8XZirKJTwac7bydFVIBbHMxUStGhd62G1s9Afi+6VMrtJniLsq4KWON9yUB2ivf0BS9kwt/Aema6v8yi14rt2Xelub8HocMomW90Qe36JZZbIdFFjs1LclUXEesim/X9dsKD9X6RmB+WBsQZVUh0wNa9n4JglBXHPhrGe6e8MBQWinHPJPjto5KbuXGH1ZFc52hGPCbvkKz1l5tfwjn9XoaEpPEdAHoYpmty9j/zUG38yEXOjgMIdwW4ilZgvJ6/+zkNnMcDTgHYY51lge+R3PBPW7Tx6tZasbXE0wA96adBWCwwYfJ7Yd+ugmEwOEwcVORKk5PV8RgiWEP kpali@hadoop-a
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCggtXsdW9p4F1fWyr+N14S43DZEE9MIATzl5cIao5pvA+rVxMz/dTvosDDwiTIJOlwTfX6qg0MVGMTXiiB/6R45v2gB75uSidFNdoblWf/wMf0svrSu6oZ7PcbgiiPyokBBhEQ39s+tU+CG3yfafpDNgWqLe0UE2V2eXIQNmv+Y2Xjl3Gj9PdwBdkQJYBC+9fZ+GUmcQVdLxZUI4D7gEAZ6Y6OeHDABaH+RHXFNk+yNdlklYKJ5QpBmMJmKEAEkA36mkXRXjbldlCpR/KZEarqRhez4gzsjiB1oM1Za4e7JJQoR5t5t62VytXJvn9z0g7DOSxnlhQ5N3Xp8PD+9BY9 kpali@hadoop-b
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxnGMtmh/YFCtubKBy7k3cZQ4yzOWAlsS9KwaOz0EsL/djRKeREi50yxgFxm/Ht/rXlQzwdV3suv3eFjWbqHDXHyJ5Q1wItoUNEc7SpDAUuqdmTznh2DbpMFUTcO3czLFtwumsqINzwGlQdd/6hS2QdCHrT/w8jV1lH4TjdvgiSy5CKvgVRJzN2RlvaO9dgl+NOQBhUbcKm77bHjfBXizIqYWoPZtNh6z6cy6Iodg8qaZpM1eVBkeHN2vcF7cszOhZDEu5fuRqm1oBo7J4DZXuVHXXFmYy+FYrIvBoU4hqiTB1M9uZskkxdAsx6pes3S8PrpJMvOmqyQJ0OQ6u+oQz kpali@hadoop-c
```

修改`authorized_keys` 文件权限：

``` shell
$ chmod 0600 ~/.ssh/authorized_keys
```

将`authorized_keys` 文件复制到另外两台主机的同一目录下：

``` shell
$ scp ~/.ssh/authorized_keys hadoop-b:~/.ssh/
$ scp ~/.ssh/authorized_keys hadoop-c:~/.ssh/
```

这样三台服务器分别就可以免密码SSH登录本机以及另外两台服务器。

### 安装Hadoop

从[官网](https://hadoop.apache.org/releases.html)下载安装包`hadoop-2.9.2.tar.gz`，分别安装到3个节点。

解压到当前目录下：

```shell
$ tar -xvf hadoop-2.9.2.tar.gz
```

使用`root`用户移动刚解压的hadoop-2.9.2`目录到`/opt`目录下：

```shell
# mv hadoop-2.9.2 /opt/
```

修改`.bash_profile`，主要是新增`HADOOP_HOME`环境变量，参考如下：

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre
export HADOOP_HOME=/opt/hadoop-2.9.2
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

使环境变量生效：

``` shell
$ source ~/.bash_profile
```

### 配置Hadoop

三个节点分别创建必要的目录：

```shell
$ cd /opt/hadoop-2.9.2
$ mkdir -p hdfs/name hdfs/data hdfs/tmp
```

然后在`hadoop-a`节点，修改`/opt/hadoop-2.9.2/etc/hadoop`目录下的配置文件，主要修改以下配置文件：

```
hadoop-env.sh
yarn-env.sh
slaves
core-site.xml
hdfs-site.xml
mapred-site.xml
yarn-site.xml
```

xml配置文件的说明和默认值可以参考以下链接：

[core-site.xml](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/core-default.xml)

[hdfs-site.xml](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)

[mapred-site.xml](http://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)

[yarn-site.xml](http://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)

修改`hadoop-env.sh`中的`JAVA_HOME`：

```
# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre
```

修改`yarn-env.sh`中的`JAVA_HOME`：

```
# some Java parameters
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/jre
```

修改`slaves`文件，修改为以下内容：

```
hadoop-b
hadoop-c
```

修改`core-site.xml`文件，配置为：

```xml
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://hadoop-a:9000</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop-2.9.2/hdfs/tmp</value>
    </property>
</configuration>
```

修改`hdfs-site.xml`文件，配置为：

``` xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop-2.9.2/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop-2.9.2/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
</configuration>
```

拷贝`mapred-site.xml.template`为`mapred-site.xml`：

``` shell
$ cp /opt/hadoop-2.9.2/etc/hadoop/mapred-site.xml.template /opt/hadoop-2.9.2/etc/hadoop/mapred-site.xml
```

修改`mapred-site.xml`文件，配置为：

``` xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

修改`yarn-site.xml`文件，配置为：

``` xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-a</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

拷贝配置文件到另外两个节点上：

``` shell
$ scp -r /opt/hadoop-2.9.2/etc/hadoop hadoop-b:/opt/hadoop-2.9.2/etc/
$ scp -r /opt/hadoop-2.9.2/etc/hadoop hadoop-c:/opt/hadoop-2.9.2/etc/
```

### 启动Hadoop

启动Hadoop之前要先格式化，在NameNode也就是`hadoop-a`上执行：

```shell
$ hdfs namenode -format
```

在`hadoop-a`上启动HDFS：

```shell
$ start-dfs.sh
```

在`hadoop-a`上启动YARN：

```shell
$ start-yarn.sh
```

在`hadoop-a`上执行`jps`命令，查看Hadoop相关进程：

```shell
$ jps | grep -v Jps
5387 ResourceManager
5672 SecondaryNameNode
5281 NameNode
```

在`hadoop-b`上执行`jps`命令，查看Hadoop相关进程：

```shell
$ jps | grep -v Jps
21072 DataNode
21206 NodeManager
```

在`hadoop-c`上执行`jps`命令，查看Hadoop相关进程：

```shell
$ jps | grep -v Jps
20685 DataNode
20810 NodeManager
```

## 使用Hadoop

可以访问Web页面http://hadoop-a:50070查看或管理HDFS，访问http://hadoop-a:8088查看或管理YARN。

### Hello World

在HDFS上创建一个目录：

``` shell
$ hadoop fs -mkdir -p /test/input
```

查看创建的目录：

``` shell
$ hadoop fs -ls /test
Found 1 items
drwxr-xr-x   - kpali supergroup          0 2019-01-23 10:22 /test/input
```

创建两个文件：

``` shell
$ echo "Hello World Bye World" > file01
$ echo "Hello Hadoop Goodbye Hadoop" > file02
```

上传文件到HDFS：

``` shell
$ hadoop fs -put file0* /test/input
```

查看刚刚上传的文件：

``` shell
$ hadoop fs -ls /test/input
Found 2 items
-rw-r--r--   1 kpali supergroup         22 2019-01-23 10:22 /test/input/file01
-rw-r--r--   1 kpali supergroup         28 2019-01-23 10:22 /test/input/file02
```

运行一个Hadoop自带的示例程序`wordcount`，这是一个统计单词的MapReduce程序，代码如下：

``` java
package org.apache.hadoop.examples;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {

  // 创建一个Mapper类
  public static class TokenizerMapper 
       extends Mapper<Object, Text, Text, IntWritable>{
    
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    // 实现map函数
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      // 提取单词，以键值对<单词,1次>写入context
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
  
  // 创建一个Reducer类
  public static class IntSumReducer 
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();
    // 实现reduce函数
    public void reduce(Text key, Iterable<IntWritable> values, 
                       Context context
                       ) throws IOException, InterruptedException {
      // 统计相同键（即同一个单词）出现的次数，并以键值对<单词,出现次数>写入context
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
    if (otherArgs.length < 2) {
      System.err.println("Usage: wordcount <in> [<in>...] <out>");
      System.exit(2);
    }
    // 创建一个Job，名称为word count
    Job job = Job.getInstance(conf, "word count");
    // 设置Job关联的一些类
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    // 设置输出的键值对类型
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    // 读取并设置输入目录
    for (int i = 0; i < otherArgs.length - 1; ++i) {
      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
    }
    // 读取并设置输出目录
    FileOutputFormat.setOutputPath(job,
      new Path(otherArgs[otherArgs.length - 1]));
    // 执行并等待Job完成
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

执行这个`wordcount`程序，将结果输出到`/test/output`目录下：

``` shell
$ hadoop jar /opt/hadoop-2.9.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar
wordcount /test/input /test/output
19/01/23 10:32:25 INFO client.RMProxy: Connecting to ResourceManager at hadoop-test/192.168.232.164:8032
19/01/23 10:32:26 INFO input.FileInputFormat: Total input files to process : 2
19/01/23 10:32:27 INFO mapreduce.JobSubmitter: number of splits:2
19/01/23 10:32:27 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
19/01/23 10:32:28 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1548209185345_0001
19/01/23 10:32:29 INFO impl.YarnClientImpl: Submitted application application_1548209185345_0001
19/01/23 10:32:29 INFO mapreduce.Job: The url to track the job: http://hadoop-test:8088/proxy/application_1548209185345_0001/
19/01/23 10:32:29 INFO mapreduce.Job: Running job: job_1548209185345_0001
19/01/23 10:32:41 INFO mapreduce.Job: Job job_1548209185345_0001 running in uber mode : false
19/01/23 10:32:41 INFO mapreduce.Job:  map 0% reduce 0%
19/01/23 10:33:08 INFO mapreduce.Job:  map 100% reduce 0%
19/01/23 10:33:24 INFO mapreduce.Job:  map 100% reduce 100%
19/01/23 10:33:26 INFO mapreduce.Job: Job job_1548209185345_0001 completed successfully
19/01/23 10:33:26 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=79
                FILE: Number of bytes written=595384
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=262
                HDFS: Number of bytes written=41
                HDFS: Number of read operations=9
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
        Job Counters
                Launched map tasks=2
                Launched reduce tasks=1
                Data-local map tasks=2
                Total time spent by all maps in occupied slots (ms)=46680
                Total time spent by all reduces in occupied slots (ms)=13499
                Total time spent by all map tasks (ms)=46680
                Total time spent by all reduce tasks (ms)=13499
                Total vcore-milliseconds taken by all map tasks=46680
                Total vcore-milliseconds taken by all reduce tasks=13499
                Total megabyte-milliseconds taken by all map tasks=47800320
                Total megabyte-milliseconds taken by all reduce tasks=13822976
        Map-Reduce Framework
                Map input records=2
                Map output records=8
                Map output bytes=82
                Map output materialized bytes=85
                Input split bytes=212
                Combine input records=8
                Combine output records=6
                Reduce input groups=5
                Reduce shuffle bytes=85
                Reduce input records=6
                Reduce output records=5
                Spilled Records=12
                Shuffled Maps =2
                Failed Shuffles=0
                Merged Map outputs=2
                GC time elapsed (ms)=334
                CPU time spent (ms)=2260
                Physical memory (bytes) snapshot=532660224
                Virtual memory (bytes) snapshot=6334607360
                Total committed heap usage (bytes)=307437568
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=50
        File Output Format Counters
                Bytes Written=41
```

也可以在YARN Web界面查看执行结果。

查看输出文件：

``` shell
$ hadoop fs -ls /test/output
Found 2 items
-rw-r--r--   1 kpali supergroup          0 2019-01-23 10:33 /test/output/_SUCCESS
-rw-r--r--   1 kpali supergroup         41 2019-01-23 10:33 /test/output/part-r-00000
```

查看输出结果：

``` shell
$ hadoop fs -cat /test/output/part-r-00000
Bye     1
Goodbye 1
Hadoop  2
Hello   2
World   2
```

### Hadoop流

Hadoop流提供了一个API，允许用户使用任何脚本语言写Map函数或者Reduce函数，Hadoop流的关键是，它使用UNIX标准流作为程序与Hadoop之间的接口，因此任何程序只要可以从标准输入流中读取数据并且可以写入数据到标准输出流，那么就可以通过Hadoop流使用其他语言编写MapReduce程序的Map函数或Reduce函数。

我们使用相同的输入数据，但是使用Linux的`cat`命令来做Map函数，使用`wc`命令来做Reduce函数，测试一下Hadoop流：

``` shell
$ hadoop jar /opt/hadoop-2.9.2/share/hadoop/tools/lib/hadoop-streaming-2.9.2.jar -input  /test/input -output /test/output1 -mapper /bin/cat -reducer /usr/bin/wc
```

同样，查看一下输出结果：

``` shell
$ hadoop fs -cat /test/output1/part-00000
      2       8      52
```

## 参考文档

[http://hadoop.apache.org/docs/stable/](http://hadoop.apache.org/docs/stable/)

[《Hadoop实战 第2版》](https://book.douban.com/subject/20275953/)

[《Hadoop权威指南》](https://book.douban.com/subject/27115351/)

[https://www.cnblogs.com/qingyunzong/p/8496127.html](https://www.cnblogs.com/qingyunzong/p/8496127.html)

