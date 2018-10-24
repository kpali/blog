---
layout: post
title: 数据库审核工具Themis的安装与试用
category: 工具
tags: Themis
---

## 目的

最近需要一款数据库审核产品，用于辅助分析数据库质量，形成报告，辅助运维工作，促进改善数据库质量。优先选择了开源产品Themis（由宜信DBA团队开发），认为初步满足需求，但由于该项目已中止开发，可能需要二次开发。

因此，通过本次测试，希望达到以下2个目的：

1. 熟悉数据库审核产品，熟悉Themis架构及其功能。
2. 评估二次开发的可行性。

## 安装Themis

### 安装操作系统CentOS 7

**安装过程**

略，操作系统默认已安装python2.7版本

**切换至root用户**

```
$ su
```

### 关闭防火墙（非必要）

**关闭防火墙**

```
# systemctl stop firewalld.service
```

**禁止防火墙开机启动**

```
# systemctl disable firewalld.service
```

### 关闭SELinux（非必要）

**修改/etc/selinux/config 文件**

将SELINUX=enforcing改为SELINUX=disabled

**重启机器**

```
# reboot
```

### 安装gcc

```
# yum install gcc
```

### 安装gcc-c++

```
# yum install gcc-c++
```

### 安装MySQL

**下载MySQL的repo源**

```
# wget http://repo.mysql.com/mysql-community-release-el7-10.noarch.rpm
```

**安装mysql-community-release-el7-10.noarch.rpm**

```
# rpm -ivh mysql-community-release-el7-10.noarch.rpm
```

**更新yum缓存**

```
# yum clean all

# yum makecache
```

**安装mysql-server**

```
# yum install mysql-server
```

**安装mysql-devel**

```
# yum install mysql-devel
```

**启动MySQL**

```
# systemctl start mysqld.service
```

**获得初始密码**

```
# grep 'temporary password' /var/log/mysqld.log
```

**使用初始密码登录**

```
# mysql -u root -p
```

> 登录时有可能报这样的错：
>
> `ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)`
>
> 原因是/var/lib/mysql的访问权限问题，下面的命令把/var/lib/mysql的拥有者改为当前用户：
>
> `# chown -R root:root /var/lib/mysql`

**更改初始密码为root（为了简单，可根据实际需要设置密码）**

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```

### 安装redis

**下载安装包**

```
# wget http://download.redis.io/releases/redis-4.0.8.tar.gz
```

**解压安装**

```
# tart -zxvf redis-4.0.8.tar.gz

# cd redis-4.0.8

# make

# make install
```

**设置redis密码**

打开/etc/redis/6379.conf，找到# requirepass foobared，去掉#号，修改密码foobared为root（为了简单，可根据实际需要设置密码），如下

> requirepass root

保存文件

**启动redis服务**

```
# systemctl start redis_6379.service
```

### 安装mongodb

**下载安装包**

```
# wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.2.tgz
```

**解压安装**

```
# tar -zxvf mongodb-linux-x86_64-rhel70-3.6.2.tgz

# mv mongodb-linux-x86_64-rhel70-3.6.2 /usr/local/mongodb
```

**配置环境变量**

修改/etc/profile，在文件最后加上

>  export MONGODB_HOME=/usr/local/mongodb
>
> export PATH=\$PATH:\$MONGODB_HOME/bin

使配置文件生效

```
# source /etc/profile
```

**创建数据文件目录**

```
# mkdir -p /usr/local/mongodb/data/db
```

**创建日志文件目录**

```
# mkdir -p /usr/local/mongodb/logs
```

**创建启动配置文件**

新建/usr/local/mongodb/bin/mongodb.conf，包含以下内容

> dbpath = /usr/local/mongodb/data/db
>
> logpath = /usr/local/mongodb/logs/mongodb.log
>
> port = 27017
>
> fork = true

**创建mongodb启动服务配置**

新建/lib/systemd/system/mongodb.service，包含以下内容

> [Unit]
>
> Description=mongodb
>
> After=network.target remote-fs.target nss-lookup.target
>
> [Service]
>
> Type=forking
>
> ExecStart=/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongodb.conf
>
> ExecReload=/bin/kill -s HUP $MAINPID
>
> ExecStop=/usr/local/mongodb/bin/mongod --shutdown --config /usr/local/mongodb/bin/mongodb.conf
>
> PrivateTmp=true
>
> [Install]
>
> WantedBy=multi-user.target  

**设置mongodb.service权限**

```
# chmod 754 /lib/systemd/system/mongodb.service
```

**启动mongodb服务**

```
# systemctl start mongodb.service
```

**创建数据库和配置用户**

```
# mongo

> use sqlreview

> db.createUser({user:"sqlreview", pwd:"sqlreview", roles:[{role:"dbOwner", db:"sqlreview"}]});
```

### 安装pip

**下载安装包**

进入[https://pypi.python.org/pypi/pip](https://pypi.python.org/pypi/pip) 下载pip-9.0.1.tar.gz

**解压安装**

```
# tar -zxvf pip-9.0.1.tar.gz `

# python setup.py install
```

### 安装cx_Oracle

```
# python -m pip install cx_Oracle --upgrade
```

### 安装Oracle Instant Client

**下载安装包**

进入[http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](https://link.jianshu.com?t=http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html) 下载

oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm

oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm

**安装**

```
# rpm -ivh oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm

# rpm -ivh oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm
```

**配置环境变量**

修改/etc/profile，追加以下内容

> export ORACLE_HOME=/usr/lib/oracle/12.2/client64
>
> export PATH=\$PATH:$ORACLE_HOME/bin
>
> export LD_LIBRARY_PATH=$ORACLE_HOME/lib
>
> export TNS_ADMIN=$ORACLE_HOME/network/admin

使配置生效

```
# source /etc/profile
```

### 安装virtualenv

```
# pip install virtualenv
```

### 安装git

```
# yum install git
```

### 新建用户（建议）

```
# adduser thmis-test

# su - themis-test
```

### 下载Themis代码

```
$ git clone https://github.com/CreditEaseDBA/Themis
```

### 创建并初始化虚拟环境

```
$ cd /home/themis-test

$ virtualenv python-project --python=python2.7

$ source /home/themis-test/python-project/bin/activate
```

### 安装其他依赖

**进入源代码目录**

```
$ cd /home/themis-test/Themis
```

**下载安装**

```
$ pip install -r requirement.txt
```

### 安装PyH

**下载安装包**

下载地址：[https://github.com/hanxiaomax/pyh](https://github.com/hanxiaomax/pyh)，下载pyh-master.zip到任意目录

**解压安装**

```
$ unzip pyh-master.zip

$ cd pyh-master

$ python setup.py install
```

## 配置Themis

### 修改Themis配置文件

**配置文件说明**

配置文件路径：/home/themis-test/Themis/settings.py，内容如下：

```
# # set oracle ipaddress, port, sid, account, password
# ipaddres : port -> key
ORACLE_ACCOUNT = {
    # oracle
    "127.0.0.1:1521": ["cedb", "system", "password"]
}

# set mysql ipaddress, port, account, password
MYSQL_ACCOUNT = {
    "127.0.0.1:3307": ["mysql", "user", "password"]
}

# pt-query save data for mysql account, password
PT_QUERY_USER = "user"
PT_QUERY_PORT = 3306
PT_QUERY_SERVER = "127.0.0.1"
PT_QUERY_PASSWD = "password"
PT_QUERY_DB = "slow_query_log"

# celery setting
REDIS_BROKER = 'redis://:password@127.0.0.1:6379/0'
# REDIS_BROKER = 'redis://:@127.0.0.1:6379/0'


REDIS_BACKEND = 'redis://:password@127.0.0.1:6379/0'
# REDIS_BACKEND = 'redis://:@127.0.0.1:6379/0'


CELERY_CONF = {
    "CELERYD_POOL_RESTARTS": True
}

# mongo server settings
MONGO_SERVER = "127.0.0.1"
MONGO_PORT = 27017
# MONGO_USER = "sqlreview"
MONGO_USER = "sqlreview"
# MONGO_PASSWORD = ""
MONGO_PASSWORD = "sqlreview"
MONGO_DB = "sqlreview"

# server port setting
SERVER_PORT = 7000

# capture time setting
CAPTURE_OBJ_HOUR = "18"
CAPTURE_OBJ_MINUTE = 15
CAPTURE_OTHER_HOUR = "18"
CAPTURE_OTHER_MINUTE = 30
```

ORACLE_ACCOUNT和MYSQL_ACCOUNT是我们需要审核的目标机器的帐号和密码，主要是在数据采集部分和对象类审核以及mysql的执行计划类审核部分会用到，因此该帐号因该具有较高的权限，为了安全在生产环境应该设置专有的帐号并设置专有的权限，或者加上一些ip的限制等

PT_QUERY_USER、PT_QUERY_PORT、PT_QUERY_SERVER、PT_QUERY_PASSWD、PT_QUERY_DB是我们pt-query-digest工具解析目标机器的慢sql后需要存储到的mysql数据库的一些配置

REDIS_BROKER、REDIS_BACKEND、CELERY_CONF是任务调度工具celery的配置选项

MONGO_SERVER、MONGO_PORT、MONGO_USER、MONGO_PASSWORD、MONGO_DB是需要存储结果集的mongo的配置选项

SERVER_PORT是web管理端监听的端口，不要使用9000和5555端口，这两个被分配给了文件下载服务器和flower管理工具

CAPTURE_OBJ_HOUR、CAPTURE_OBJ_MINUTE、CAPTURE_OTHER_HOUR、CAPTURE_OTHER_MINUTE是针对oracle的数据采集模块需要设置的采集时间，根据自己的实际情况设置不同的时间即可，避开业务高峰期

**修改配置**

我们现在仅需要审核Oracle数据库，IP地址是10.168.6.190，实例是ora11g，因此修改ORACLE_ACCOUNT如下：

```
ORACLE_ACCOUNT = {
    # oracle
    "10.168.6.190:1521": ["ora11g", "system", "oracle"]
}
```

修改REDIS_BROKER和REDIS_BACKEND如下：

```
REDIS_BROKER = 'redis://:root@127.0.0.1:6379/0'
REDIS_BACKEND = 'redis://:root@127.0.0.1:6379/0'
```

修改MONGO配置

```
MONGO_SERVER = "127.0.0.1"
MONGO_PORT = 27017
MONGO_USER = ""
MONGO_PASSWORD = ""
MONGO_DB = "sqlreview"
```

修改采集时间，根据实际需要设置

```
CAPTURE_OBJ_HOUR = "12"
CAPTURE_OBJ_MINUTE = 15
CAPTURE_OTHER_HOUR = "12"
CAPTURE_OTHER_MINUTE = 30
```

### 规则导入

```
$ cd /home/themis-test/Themis
```

```
$ mongoimport -h 127.0.0.1 --port 27017 -u sqlreview -p sqlreview -d sqlreview -c rule --file script/rule.json
```

###  要审核的Oracle机器依赖安装

无

### 要审核的MySQL机器依赖安装

安装pt-query-digest

### 启动Themis

```
$ cd /home/themis-test/Themis
```

```
$ supervisord -c script/supervisord.conf
```

访问http://127.0.0.1:7000

## 试用Themis

Themis启动后，数据采集根据之前的配置是每天采集一次。

### 主界面

进入主界面，从导航栏可以看到，功能不多，只有4个：规则编辑、规则增加、发布任务、任务详情。

### 规则编辑

主界面即是规则编辑页面，由于是初次使用，基本上都是内置规则（共112项），在此页面可以开启关闭指定规则、设置权重等等。

![规则设置](/assets/img/2018-02-14-install-themis/规则设置.jpg)

内置规则说明：

![规则_对象类_Oracle](/assets/img/2018-02-14-install-themis/规则_对象类_Oracle.jpg)

![规则_对象类_MySQL](/assets/img/2018-02-14-install-themis/规则_对象类_MySQL.jpg)

![规则_执行计划类_Oracle](/assets/img/2018-02-14-install-themis/规则_执行计划类_Oracle.jpg)

![规则_执行计划类_MySQL](/assets/img/2018-02-14-install-themis/规则_执行计划类_MySQL.jpg)

![规则_执行特征类](/assets/img/2018-02-14-install-themis/规则_执行特征类.jpg)

![规则_文本类](/assets/img/2018-02-14-install-themis/规则_文本类.jpg)

### 规则增加

分为简单规则和复杂规则，可以添加自定义的规则。

![简单规则增加](/assets/img/2018-02-14-install-themis/简单规则增加.jpg)

![复杂规则增加](/assets/img/2018-02-14-install-themis/复杂规则增加.jpg)

### 任务发布

指定要审核的数据库地址、规则类型、日期等，点击执行任务即可执行，任务的执行是异步的，一般在5分钟内完成。

![任务发布](/assets/img/2018-02-14-install-themis/任务发布.jpg)

### 任务详情

即查看任务发布执行的结果。

![任务详情1](/assets/img/2018-02-14-install-themis/任务详情1.jpg)

选择其中一条任务，点击查看报告，即可查看该任务的详情报告。

![任务详情2](/assets/img/2018-02-14-install-themis/任务详情2.jpg)

## 结论

### Themis架构及其功能

![架构](/assets/img/2018-02-14-install-themis/架构.png)

#### 架构概述

**进程监控和管理：**

- 使用python编写的supervisor，实现Themis所需服务或进程的监控和自启动等。

**数据采集：**

- 如需审核Oracle数据库，通过在Themis所在机器安装Oracle Instant Client，并使用python编写的cx_Oracle来连接Oracle数据库并采集数据。
- 如需审核MySQL数据库，通过在目标机器安装慢查询分析工具pt-query-digest，并使用Themis所在机器的MySQL数据库存储pt-query-digest的分析结果。

**调度:**

- 使用python编写的celery，实现任务调度，并使用redis用来作为celery的调度队列。

**存储：**

- Themis使用Mongodb存储采集到的数据、审核规则、任务执行结果等数据。

**审核引擎：**

- 由Themis使用python编写。

**Web界面：**

- 使用了python模块pyH生成HTML内容。
- 除了html、js、css等，其他均为python编写。

#### 功能概述

由于Themis只完成第一期的开发便中止了，因此当前具备的功能如下：

1. 通过WEB界面完成全部工作，主要使用者是DBA和有一定数据库基础的研发人员。
2. 可针对某个用户审核，可审核包括数据结构、SQL文本、SQL执行特征、SQL执行计划等多个维度。
3. 审核结果通过WEB页面或导出文件的形式提供。
4. 支持Oracle（10g及以上）、MySQL（5.6及以上）数据库。
5. 可以自定义审核规则。

### 二次开发可行性评估

Themis主要使用python开发，估计代码量在5000行以内，规模一般。

Themis安装过程所需的组件和模块较多较杂，部署上有一定难度，如果二次开发可考虑简化。

**Themis内置的112项审核规则是其核心价值所在，且可以直接提取使用**，其中Oracle规则共69项，MySQL规则共43项。

因此，进行二次开发的难度不高，但鉴于Themis当前版本功能较少、规模较小，后续还需结合实际场景，来综合评估开发价值。

## 参考文档

Themis的github仓库：[https://github.com/CreditEaseDBA/Themis/tree/master](https://github.com/CreditEaseDBA/Themis/tree/master)

Themis的gitbook文档：[https://tuteng.gitbooks.io/themis/content/](https://tuteng.gitbooks.io/themis/content/)