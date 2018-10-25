---
layout: post
title: 安装Arch Linux
description: “Arch Linux(或称Arch)是一种以轻量简洁为设计理念的Linux发行版。Arch Linux的哲学就是Keep It Simple, Stupid。遵循简洁、现代、实用、以用户为中心等核心原则。 同时Arch Linux还拥有相当之完善的Wiki文档，在安装和使用的过程中，大多数问题都能得到解决。”
category: 工具
tags: ArchLinux
---

## Arch Linux

Arch Linux(或称Arch)是一种以轻量简洁为设计理念的Linux发行版。Arch Linux的哲学就是Keep It Simple, Stupid。遵循简洁、现代、实用、以用户为中心等核心原则。

同时Arch Linux还拥有相当之完善的Wiki文档，在安装和使用的过程中，大多数问题都能得到解决。

## 用fdisk建立分区

创建一个48G的分区，和一个2G的swap分区。输入fdisk /dev/sda启动fdisk，fdisk基本命令如下：

```
n：创建新分区

d：删除一个分区

p：预览分区表

a：设置启动分区

w：写入分区表

q：退出
```

### 启动fdisk

	# fdisk /dev/sda

### 创建分区表

```
Command (m for help): 
```

输入o并按下Enter

**创建第一个分区**

```
Command (m for help): 
```

输入n并按下Enter

```
Partition type：Select (default p):
```

按下Enter

```
Partition number (1-4), default 1):
```

按下Enter

```
First sector (2048-104857599, default 2048:
```

按下Enter

```
Last sector, +sectors or +size{K,M,G,T,P} (2048-104857599,default 104857599):
```

输入+48G并按下Enter

**然后建立第二个分区**

```
Command (m for help):
```

 输入n并按下Enter

```
Partition type：Select (default p):
```

按下Enter

```
Partition number (2-4), default 2):
```

按下Enter

```
First sector (100663291-104857599, default 100665344:
```

按下Enter

```
Last sector, +sectors or +size{K,M,G,T,P} (100665344-104857599,default 104857599):
```

按下Enter

**设置启动分区**

```
Command (m for help): 
```

输入a并按下Enter

```
Partition number (2-4), default 2):
```

输入1按下Enter

**预览新的分区表**

```
Command (m for help): 
```

输入p并按下Enter

**然后向磁盘写入这些改动**

```
Command (m for help):
```

 输入w并按下Enter

## 创建文件系统

将48G的分区格式化为ext4，将2G的分区格式化为swap并启用

```
# mkfs.ext4 /dev/sda1

# mkswap /dev/sda2

# swapon /dev/sda2
```

挂载分区，将根分区挂载到/mnt

```
# mount /dev/sda1 /mnt
```

## 选择安装镜像

安装前需要编辑/etc/pacman.d/mirrorlist，将偏好的镜像放到最前面，或者只使用一个镜像并删光其他行，但为保险，还是留其他几个离您较近的镜像作备用好。mirrorlist文件也会被pacstrap复制到新系统，所以最好现在就设置。

设置完成后使用pacman -Syy强制刷新

## 使用pacstrap安装基本系统

```
# pacstrap -i /mnt base base-devel
```

## 生成fstab

用以下命令生成fstab，之所以用UUID时因为它们能唯一且独立地标识，如果您想用卷标，用-L代替-U即可。

警告：强烈建议在生成fstab后检查一下是否正确。若在运行genfstab或是之后发生错误，请勿再次运行genfstab，而是直接手动编辑fstab文件。

```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```

## chroot并开始配置新系统

```
# arch-chroot /mnt /bin/bash
```

从现在开始，我们会通过编辑文件以配置系统。所以若文件不存在，手动创建，或者您也可以加以编辑已存在的文件，以修改默认参数。

### Locale

/etc/locale.gen是一个仅包含注释文档的文本文件。指定您需要的本地化类型，只需移除对应行前面的注释符号（＃）即可，请选择带UTF-8的项：

```
# vi /etc/locale.gen
```

```
en_US.UTF-8 UTF-8

zh_CN.UTF-8 UTF-8

zh_TW.UTF-8 UTF-8
```

接着执行locale-gen以生成locale讯息

```
# locale-gen
```

创建/etc/locale.conf并提交您的本地化选项

```
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```

警告：不推荐在此设置任何中文locale，或导致tty乱码。

提示：所提交的LANG变量，要事先在/etc/locale.gen反注释好

### 时区

将/etc/localtime软链接到/usr/share/zoneinfo/Zone/SubZone，以上海为例：

```
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 硬件时间

使用UTC

```
# hwclock --systohc --utc
```

### 主机名

设置个您喜欢的主机名，例如：

```
# echo Arch > /etc/hostname
```

并在/etc/hosts添加同样的主机名：

```
#vi /etc/hosts

#
# /etc/hosts: static lookup table for host names
#

#<ip-address> <hostname.domain.org><hostname>
127.0.0.1 localhost.localdomainlocalhostArch
::1 localhost.localdomainlocalhost

# End of file
```

### 安装net-tools以使用ipconfig命令

```
# pacman -S net-tools
```

### 使用dhcpcd配置动态IP有线网络

systemctl enable dhcpcd@interface_name.service

interface_name使用ipconfig命令得interface_name为ens33

```
# systemctl enable dhcpcd@ens33.service
```

### 设置root密码

```
# passwd
```

### 添加普通用户

> useradd命令主要参数：
>
> －c：加上备注文字，备注文字保存在passwd的备注栏中。
>
> －d：指定用户登入时的启始目录。
>
> －D：变更预设值。
>
> －e：指定账号的有效期限，缺省表示永久有效。
>
> －f：指定在密码过期后多少天即关闭该账号。
>
> －g：指定用户所属的群组。
>
> －G：指定用户所属的附加群组。
>
> －m：自动建立用户的登入目录。
>
> －M：不要自动建立用户的登入目录。
>
> －n：取消建立以用户名称为名的群组。
>
> －r：建立系统账号。
>
> －s：指定用户登入后所使用的shell。
>
> －u：指定用户ID号。
>

```
# useradd -m -G wheel -s /bin/bash user

# passwd user
```

提示：user为你要使用的用户名

添加普通用户到/etc/sudoers，使普通用户能够使用sudo命令

```
# chmod +w /etc/sudoers

# vi /etc/sudoers
```

找到root ALL=(ALL) ALL，往后新增一行

```
user ALL=(ALL) ALL
```

```
# chmod -w /etc/sudoers
```

提示：user为你要使用的用户名

### 安装并配置GRUB

```
# pacman -S grub

# grub-install --target=i386-pc --recheck /dev/sda

# grub-mkconfig -o /boot/grub/grub.cfg
```

### 离开chroot环境并重启系统

```
# exit

# reboot
```

## 安装桌面环境

### 安装xorg-server

```
# pacman -S xorg-server xorg-server-utils xorg-xinit
```

### 安装显卡驱动

如果不知道是什么显卡，就使用以下命令查看

```
# lspci | grep VGA
```

这里使用xf86-video-vmware

```
# pacman -S mesa

# pacman -S xf86-video-vmware
```

### 测试X环境是否正常

```
# pacman -S xorg-twm xorg-xclock xterm

# startx

# exit

# pkill X
```

### 安装ALSA(声卡)

```
# pacman -S alsa-utils
```

### 调节音量

```
# alsamixer 
```

### 安装D-BUS守护进程

```
# pacman -S dbus
```

### 安装GNOME

```
# pacman -S gnome gnome-extra

# pacman -S gdm
```

### 设置桌面环境启动方式

将下面一行添加到您的~/.xinitrc文件中

```
exec gnome-session
```

现在GNOME将在您登录后使用startx时启动

```
# systemctl enable gdm.service
```

现在gdm将在您开机时自动启动

### 安装常用软件

```
# pacman -S gnome-tweak-tool

# pacman -S gvim

# pacman -S emacs

# pacman -S gimp

# pacman -S firefox

# pacman -S thunderbird

# pacman -S libreoffice
```

### 安装开发环境

```
# pacman -S monodevelop

# pacman -S eclipse

# pacman -S codeblocks
```

### 安装字体

```
# pacman -S ttf-dejavu

# pacman -S wqy-zenhei

# pacman -S wqy-microhei
```

### 安装VMware-tools（需重启）

```
# pacman -S open-vm-tools
```

### 安装VMware鼠标驱动（需重启）

```
# pacman -S xf86-input-vmmouse
```

### 安装fcitx输入法

```
# pacman -S fcitx-im fcitx-configtool fcitx-sunpinyin
```

然后添加以下三行到~/.xinitrc

```
export GTK_IM_MODULE=fcitx

export QT_IM_MODULE=fcitx

export XMODIFIERS="@im=fcitx"
```