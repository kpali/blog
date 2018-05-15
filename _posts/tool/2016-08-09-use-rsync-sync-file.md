---
layout: post
title: 使用rsync同步文件
category: 工具
tags: Rsync
---

rsync是Unix/Linux下同步文件的一个高效算法，它能同步更新两处计算机的文件与目录，并适当利用查找文件中的不同块以减少数据传输。关于rsync的核心算法，有兴趣的看下[这篇文章。](http://coolshell.cn/articles/7425.html "rsync的核心算法")

这里介绍一下使用方法，**我们假设要从A机器的/a目录同步文件到B机器的/b目录**，为了简单起见，以下操作均使用root权限。

## 安装rsync

首先是rsync的安装，Linux下一般都会自带，如果没有就用包管理工具安装，或者到[rsync官网](https://rsync.samba.org/)下载手动安装；Windows下则可以安装cygwin，安装时选择包含rsync包即可。也可以选择cwRsync，cwRsync其实也是rsync基于cygwin环境在Windows上的实现，不过自4.1.0版本之后开始收费了，另外据我测试，cwRsync的性能没有比直接装cygwin来得好，非严谨测试，仅供参考。

## 创建secrets文件

我们把A机器当作服务端，B机器当作客户端。

因此在**A机器**上创建文件/etc/server.secrets，该文件相当于是rsync的用户名密码数据库。假如只有一个用户，用户名是rsyncuser，密码是rsyncpwd，文件内容如下：

	rsyncuser:rsyncpwd

修改文件的访问权限为600，rsync强制要求必须是600，否则使用时会报错。

	# chmod 600 /etc/server.secrets

然后在**B机器上**创建文件/etc/client.secrets，客户端的secrets文件只需要填写密码即可，文件内容如下：

	rsyncpwd

修改文件的访问权限为600，rsync强制要求必须是600，否则使用时会报错。

	# chmod 600 /etc/server.secrets

## 创建rsyncd.conf文件

在**A机器**上创建文件/etc/rsyncd.conf，该文件是rsync的配置文件，大致的格式如下：

```
全局参数

[模块名称1]
模块参数1
模块参数2

[模块名称2]
模块参数1
模块参数2
```

有些参数必须是全局参数，有些参数既可以是全局参数也可以是模块参数，根据写的位置不同而不同。

全局参数：

> port：指定后台程序使用的端口号，默认为873。
> 
> log file：指定rsync的日志文件，而不将日志发送给syslog。比如可指定为/var/log/rsyncd.log
> 
> pid file：指定rsync的pid文件，通常指定为/var/run/rsyncd.pid

模块参数：

> path：指定该模块的供备份的目录树路径，该参数是必须指定的。
>
> comment：给模块指定一个描述，该描述连同模块名在客户连接得到模块列表时显示给客户。默认没有描述定义。
>
> uid：该选项指定当该模块传输文件时守护进程应该具有的uid，配合gid选项使用可以确定哪些可以访问怎么样的文件权限，默认值是nobody
>
> gid：该选项指定当该模块传输文件时守护进程应该具有的gid。默认值为nobody
>
> use chroot：如果use chroot指定为true，那么rsync在传输文件以前首先chroot到path参数所指定的目录下。这样做的原因是实现额外的安全防护，但是缺 点是需要以roots权限，并且不能备份指向外部的符号连接所指向的目录文件。默认情况下chroot值为true。
>
> max connections：指定该模块的最大并发连接数量以保护服务器，超过限制的连接请求将被告知随后再试。默认值是0，也就是没有限制。
>
> list：该选项设定当客户请求可以使用的模块列表时，该模块是否应该被列出。如果设置该选项为false，可以创建隐藏的模块。默认值是true。
>
> hosts allow：该选项指定哪些IP的客户允许连接该模块。客户模式定义可以是以下形式： 单个IP地址，例如：192.167.0.1
> 
> read only：该选项设定是否允许客户上载文件。如果为true那么任何上载请求都会失败，如果为false并且服务器目录读写权限允许那么上载是允许的。默认值为true。
>
> hosts deny：指定不允许连接rsync服务器的机器，可以使用hosts allow的定义方式来进行定义。默认是没有hosts deny定义。
> 
> transfer logging：使rsync服务器使用ftp格式的文件来记录下载和上载操作在自己单独的日志中。
> 
> auth users：该选项指定由空格或逗号分隔的用户名列表，只有这些用户才允许连接该模块。这里的用户和系统用户没有任何关系。如果auth users被设置，那么客户端发出对该模块的连接请求以后会被rsync请求challenged进行验证身份这里使用的 challenge/response认证协议。用户的名和密码以明文方式存放在secrets file选项指定的文件中。默认情况下无需密码就可以连接模块(也就是匿名方式)。
> 
> secrets file：该选项指定一个包含定义用户名:密码对的文件。只有在”auth users”被定义时，该文件才有作用。文件每行包含一个username:passwd对。一般来说密码最好不要超过8个字符。没有默认的 secures file名，需要限式指定一个(例如：/etc/rsyncd.passwd)。注意：该文件的权限一定要是600，否则客户端将不能连接服务器。
> 
> strict modes：该选项指定是否监测密码文件的权限，如果该选项值为true那么密码文件只能被rsync服务器运行身份的用户访问，其他任何用户不可以访问该文 件。默认值为true。

这里我们编写rsyncd.conf内容如下：

```
uid = root
gid = root
use chroot = false
hosts allow = *
log file = /var/log/rsyncd.log
read only = false
transfer logging = yes
secrets file = /etc/server.secrets
auth users = rsyncuser
[a]
path = /a/
```

## 启动rsync守护进程

在**A机器**上执行：

	# /usr/bin/rsync --daemon

## 开始同步

在**B机器**执行：

	# rsync -ahvP --password-file=/etc/client.secrets 'rsyncuser@192.168.1.100::a' '/b/'

如果需要压缩，则添加-z选项：

	# rsync -ahvPz --password-file=/etc/client.secrets 'rsyncuser@192.168.1.100::a' '/b/'

如果需要限速，则添加--bwlimit选项，单位是KB：

	# rsync -ahvPz --password-file=/etc/client.secrets --bwlimit 200 'rsyncuser@192.168.1.100::a' '/b/'

如果只需要列出差异的文件，不想真正的传输文件，则添加--dry-run选项：

	# rsync -ahvPz --password-file=/etc/client.secrets --dry-run 'rsyncuser@192.168.1.100::a' '/b/'

如果需要在执行结尾列出同步结果，则添加--stats选项：

	# rsync -ahvPz --password-file=/etc/client.secrets --stats 'rsyncuser@192.168.1.100::a' '/b/'

## 总结

以上便是rsync一次文件同步的简单示例。