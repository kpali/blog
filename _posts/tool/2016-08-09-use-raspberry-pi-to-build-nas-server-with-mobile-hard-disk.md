---
layout: post
title: 用树莓派和移动硬盘搭建NAS服务器
category: 工具
tags: RaspberryPi
---

由于树莓派的USB接口不足以给移动硬盘供电，因此需要另外给移动硬盘提供电源。

### 显示当前已有的存储设备

```
# fdisk -l
```

```
Disk /dev/mmcblk0: 7876 MB, 7876902912 bytes
4 heads, 16 sectors/track, 240384 cylinders, total 15384576 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000b5098

Device Boot         Start     End          Blocks    Id   System
/dev/mmcblk0p1      8192      122879       57344     c    W95 FAT32 (LBA)
/dev/mmcblk0p2      122880    15384575     7630848   83   Linux

Disk /dev/sda: 1000.2 GB, 1000204795904 bytes
255 heads, 63 sectors/track, 121601 cylinders, total 1953524992 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x26918b4f

Device     Boot      Start  End          Blocks       Id   System
/dev/sda1  *         64     1953520064   976760000+   7    HPFS/NTFS/exFAT
```

/dev/mmc是树莓派系统的分区，mmc指的是SD卡，/dev/sda1是插上去的移动硬盘

### 安装ntfs-3g模块，以能够读写NTFS格式的硬盘

	# apt-get install ntfs-3g

### 创建一个目录，并以这个目录作为挂载点挂载硬盘

	# mkdir -p /share/disk1
	
	# chown pi.pi /share/disk1
	
	# mount -t auto /dev/sda1 /share/disk1

### 解决树莓派重启后驱动器的挂载失效的问题，任选一个

**1.安装autofs**
	
	# apt-get install autofs

编辑配置文件

	# vi /etc/auto.master

在+auto.master下面加入一行

	/share /etc/auto.ext-usb --timeout=10,defaults,user,exec,uid=1000

**2.编辑/etc/fstab**

	# vi /etc/fstab

加入一行

	/dev/sda1   /share/disk1    ntfs    defaults    0    0

**3.将mount命令加入~/.profile中**

	mount -t auto /dev/sda1 /share/disk1

### Samba的安装和配置

	# apt-get install samba samba-common-bin

备份配置文件
	
	# cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

编辑配置文件
	
	# vi /etc/samba/smb.conf

查找#  security = user，去掉这一行前面的注释符号#

	security = user

移动到文本末尾，添加网络共享，然后保存退出

	[disk1]
	path = /share/disk1
	valid users = @users
	force group = users
	create mask = 0660
	directory mask = 0771
	read only = no

然后重启Samba

	# service samba restart

将用户加入到Smaba中，这里以用户pi为例

	smbpasswd -a pi

至此，便可以使用其他机器访问树莓派上共享的文件

### DLNA的安装与配置

安装minidlna

	# apt-get install minidlna

编辑配置文件

	# vi /etc/minidlna.conf

在文件末尾加入以下内容

	#媒体文件目录
	media_dir=A,/share/DLNA/Music
	media_dir=P,/share/DLNA/Picture
	media_dir=V,/share/DLNA/Video
	#数据库目录，minidlna使用的是sqlite数据库来索引文件
	db_dir=/share/DLNA/db
	#日志目录
	log_dir=/share/DLNA/log
	#服务器IP
	listening_ip=192.168.1.120
	#端口
	port=8200
	#网络名称，用于其他设备发现当前设备
	friendly_name=RaspberryPi

然后建立以上用到的各个目录

可以选择让minidlna随机启动

	# update-rc.d minidlna defaults

取消minidlna开机自动启动

	# update-rc.d -f minidlna remove

启动minidlna服务

	# service minidlna start

停止minidlna服务

	# service minidlna stop

停止minidlna所有进程

	# killall minidlna

重启minidlna

	# service minidlna restart

查看minidlna状态
	
	# service minidlna stauts

修改配置或媒体资源更新时，需要强制刷新，以便minidlna对最新的媒体文件进行索引

	# service minidlna force-reload

卸载minidlna

	# apt-get remove --purge minidlna

通过浏览器查看资源个数

	http://192.168.1.120:8200/

在Windows操作系统的机器上，会多出一个媒体设备，假如/share/DLNA/Music目录中有音乐文件，然后点击这个媒体设备，Windows Media Player会启动，在左侧菜单中选择 其他媒体库->树莓派，然后选择音乐，就可以播放了。
注：图片格式不能为PNG

------------------------------------------------------------------------

文章参考自：http://linux.cn/article-1745-1-weixin.html

文章参考自：http://www.eeboard.com/bbs/thread-27434-1-1.html

文章参考自：http://www.eeboard.com/bbs/thread-27431-1-1.html

文章参考自：http://www.eeboard.com/bbs/thread-27399-1-1.html