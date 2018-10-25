---
layout: post
title: Git常用命令
description: "本文整理了一些常用的Git命令，用于平时遇到某些生疏操作的时候查阅。"
category: 工具
tags: Git
---

本文整理了一些常用的Git命令，用于平时遇到某些生疏操作的时候查阅。

### 创建

克隆一个远程仓库

	git clone 用户@服务器地址:仓库名.git

在当前目录新建一个Git仓库

	git init

新建一个目录，将其初始化为Git仓库

	git init 仓库名

用于在服务器上新建一个目录，将其初始化为一个裸仓库

	git init --bare 仓库名.git

### 配置

配置用户名

	git config --global user.name "用户名"

配置电子邮箱

	git config --global user.email "电子邮箱"

### 本地修改

查看工作区修改状态

	git status

添加所有修改

	git add .

提交所有修改

	git commit -m "修改说明"

### 查看历史

查看提交日志

	git log

查看提交日志和修改的文件

	git log --stat

查看精简版提交日志和图形展示

	git log --graph --pretty=oneline --abbrev-commit

查看命令历史

	git reflog

### 分支

查看所有分支

	git branch -a

切换分支

	git checkout 分支名

创建并切换到新分支

	git checkout -b 新分支名

在已有分支上创建分支，并切换到新分支

	git checkout -b 新分支名 已有分支名

删除本地分支

	git branch -d 分支名

删除远程分支

	git push origin --delete 分支名

### 标签

查看所有标签

	git tag

添加标签

	git tag 标签名

添加标签在指定提交id

	git tag 标签名 提交id 

删除本地标签

	git tag -d 标签名

删除远程标签

	git push origin :refs/tags/标签名

推送某个标签到远程

	git push origin 标签名

推送所有未推送到远程的标签

	git push origin --tags

获取所有远程标签

	git pull origin --tags

### 更新和推送

拉取远程分支更新并合并到当前分支

	git pull origin 远程分支名

拉取远程分支更新并合并到指定分支

	git pull origin 远程分支名:指定分支名

下载远程所有分支更新

	git fetch

下载远程所有分支更新，带修剪

	git fetch --prune

下载远程分支更新

	git fetch origin 远程分支名

推送当前分支的提交记录到远程分支

	git push origin 远程分支名

推送所有分支到远程仓库

	git push origin --all

### 合并

合并指定分支到当前分支

	git merge --no-ff 指定分支

解决冲突后标记文件为冲突已解决

	git add 文件

### 撤销

撤销暂存区的所有修改

	git reset --hard HEAD

撤销暂存区的所有修改到指定提交id

	git reset --hard 提交id

撤销工作区所有修改

	git checkout .

撤销指定提交

	git revert 提交id

### 暂存

将当前未提交的变化暂存起来

	git stash

查看暂存列表

	git stash list

恢复最近的暂存内容

	git stash pop

恢复指定暂存内容

	git stash apply stash@{编号}