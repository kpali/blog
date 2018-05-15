---
layout: post
title: 在Windows下使用Git
category: 工具
tags: Git
---

## 关于Git

git是当今最流行的版本控制系统，因为是诞生在Linux操作系统下，因此Linux对git天生有最好的支持，但好在各路大牛的努力下，目前在Windows也能较为完美的使用。以下便是我使用git上的一些经验。

## 客户端版本选择

### 命令行客户端

* Git For Windows：Git的官方客户端，优先推荐这个客户端，有安装版和免安装版，我使用的是免安装版，即PortableGit。网址：[https://git-scm.com/](https://git-scm.com/)

* Cygwin：Cygwin是一个在Windows平台上运行的类Unix模拟环境，稍微麻烦一些，下载时需要勾选devel/git和shells/bash_completion的软件包，如果你熟悉Linux/Unix环境，使用Cygwin就可以自选所需的软件包，能够增强Git的使用体验。网址：[http://www.cygwin.com/](http://www.cygwin.com)

### 图形界面客户端

* TortoiseGit：有用过SVN都很熟悉的Tortoise，需要先安装Git命令行客户端，然后安装TortoiseGit后，配置好Git程序路径才可使用，使用体验与TortoiseSVN基本一致。网址：[https://tortoisegit.org](https://tortoisegit.org)

* SourceTree：SourceTree在操作上会比Tortoise要直观简单一些，缺点是在查看分支图和新旧内容比对方面会比TortoiseGit弱一些。网址：[https://www.sourcetreeapp.com](https://www.sourcetreeapp.com)

## 中文乱码的处理

在Windows上使用Git，一般都会遇到中文乱码的问题，网上能搜到的解决方案有不少，比如以下2篇：

[《Git for windows 中文乱码解决方案》](https://segmentfault.com/a/1190000000578037)

[《GIT乱码解决方案汇总》](http://www.cnblogs.com/perseus/archive/2012/11/21/2781074.html)

我的操作系统是Window7，服务端操作系统是Linux且使用UTF-8编码，在解决中文乱码上花了不少时间，最后总结了几个解决步骤（可能仅适用于我的环境）：

1.将core.quotepath设为false，就不会对0x80以上的字符进行quote，否则提交文件时可能会显示类似\346\265\213\350\257\225\346\226\207\344\273\266.txt的乱码

	git config --global core.quotepath false

2.设置提交编码为utf-8

	git config --global i18n.commitencoding utf-8

3.设置日志打印编码为utf-8

	git config --global i18n.logoutputencoding utf-8

4.git log命令不像其它vcs一样，n条log从头滚到底，它会恰当地停在第一页，按space键再往后翻页。这是通过将log送给less处理实现的。以上即是设置less的字符编码，使得$ git log可以正常显示中文。因此编辑/etc/profile文件，添加如下一行

	export LESSCHARSET=utf-8

5.右键git-bash窗口标题栏，选择Options，找到Text，将Locale设置为zh_CN，Character set设置为UTF-8

## 在add文件时可能遇到的警告信息

	warning: LF will be replaced by CRLF

	fatal: CRLF would be replaced by LF

这是行结束符自动转换导致的，可以关闭自动转换功能来解决，关于行结束符的解释：[http://blog.csdn.net/feng88724/article/details/11600375](http://blog.csdn.net/feng88724/article/details/11600375)

	git config --global core.autocrlf false

至此，Git就可以在Windows下正常的使用了。