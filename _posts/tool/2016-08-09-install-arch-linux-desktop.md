---
layout: post
title: 安装Arch Linux桌面环境
category: 工具
tags: ArchLinux
---

## 安装基础包

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

## 安装桌面环境

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
\# pacman -S ttf-dejavu

\# pacman -S wqy-zenhei

\# pacman -S wqy-microhei
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

