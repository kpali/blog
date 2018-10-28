---
layout: post
title: 应用容器引擎Docker入门
description: "Docker是一个开源的应用容器引擎，基于Go语言，并遵从Apache2.0协议开源。 Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。"
category: 技术
tags: Docker
---

## Docker简介

Docker是一个开源的应用容器引擎，基于Go语言，并遵从Apache2.0协议开源。

Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

### Docker和虚拟机的区别

虚拟机是硬件资源的虚拟化，可模拟出一台完整的计算机和操作系统。

而Docker是操作系统级别的虚拟化环境，Docker引擎的基础是Linux容器（Linux Containers，LXC）技术，虚拟环境有自己的进程和网络空间。

Docker和虚拟机并不是相互取代的关系，Docker相比较虚拟机更轻量级，资源占用更小，启动更快，使用方式更灵活。但也有它的局限性，例如由于依赖LXC技术，所以无法在Windows或macOS等其他操作系统上直接运行，不过官方提供了Docker for Windows和Docker for Mac，虽然本质上是运行在一个基于轻量级虚拟机的Linux系统上，可以理解为自动安装了一个VMWare或VirtualBox来支持Docker，不过Docker for Windows和Docker for Mac的优点是占用资源少，图形界面管理，并且与系统自带的终端完美集成，操作上与Linux版本的Docker一致。

### Docker的基本原理

参考以下文章：

[Docker基础技术：Linux Namespace（上）](https://coolshell.cn/articles/17010.html)

[Docker基础技术：Linux Namespace（下）](https://coolshell.cn/articles/17029.html)

[Docker基础技术：Linux CGroup](https://coolshell.cn/articles/17049.html)

[Docker基础技术：AUFS](https://coolshell.cn/articles/17061.html)

[Docker基础技术：Device Mapper](https://coolshell.cn/articles/17200.html)

## Docker的核心概念

### Docker镜像

Docker镜像（Image）类似于虚拟机镜像，可以将它理解为一个面向Docker引擎的只读模板，包含了文件系统。

一个镜像可以只包含一个完整的Ubuntu操作系统环境，可以把它称为一个Ubuntu镜像。

镜像是创建Docker容器的基础，可以从网上下载一个已经做好的应用镜像，并通过简单的命令就可以直接使用。

### Docker容器

Docker容器（Container）类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。

容器是从镜像创建的应用运行实例，可以将其启动、开始、停止、删除，而这些容器都是相互隔离、互不可见的。

可以把容器看做一个简易版的Linux系统环境（这包括root用户权限、进程空间、用户空间和网络空间等）。

镜像自身是只读的。容器从镜像启动的时候，Docker会在镜像的最上层创建一个可写层，镜像本身将保持不变。

### Docker仓库

Docker仓库（Repository）类似于代码仓库，是Docker集中存放镜像文件的场所。

目前，最大的公开仓库是Docker Hub，存放了数量庞大的镜像供用户下载。

## 安装Docker CE

### 测试环境

操作系统：Ubuntu 18.04 64-bit

### 卸载旧版本

较旧版本的Docker被称为`docker`或`docker-engine`。如果安装过，则卸载它们：

```shell
$ sudo apt-get remove docker docker-engine docker.io
```

较新版本的Docker被称为`docker-ce`，Ubuntu上的Docker CE支持`overlay2`和`aufs`存储驱动程序。

- 在Linux内核版本4或更高版本上安装Docker，支持`overlay2`，并且优先于`aufs`。
- 在Linux内核版本3的版本上安装Docker，支持`aufs` ，因为该内核版本不支持`overlay`或`overlay2`。

### 使用apt包管理安装Docker

更新`apt`包索引：

```shell
$ sudo apt-get update
```

安装以下软件包以允许`apt`通过HTTPS使用仓库：

```shell
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

添加Docker的官方GPG密钥：

```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

设置Docker仓库到apt：

```shell
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

再更新apt包索引：

```shell
$ sudo apt-get update
```

安装Docker CE，默认安装最新版本（本次安装版本为`18.06.1-ce`），安装完成后Docker守护进程自动启动：

```shell
$ sudo apt-get install docker-ce
```

> 如果要安装指定版本的Docker，可以先查询可用的版本：
>
> ```shell
> $ apt-cache madison docker-ce
> ```
>
> 返回结果可能如下：
>
> ```
>  docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
>  docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
>  docker-ce | 18.03.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
> ```
>
> 选择要安装的版本，例如`18.03.1~ce3-0~ubuntu`，则在安装时指定该版本：
>
> ```shell
> $ sudo apt-get install docker-ce=18.03.1~ce~3-0~ubuntu
> ```

通过运行hello-world镜像验证Docker CE是否已正确安装：

```shell
$ sudo docker run hello-world
```

以上命令会下载测试镜像并在容器中运行它，当容器运行时，它会打印一条信息并退出。

### 使用国内Docker Hub源加速下载

Docker需要从Docker Hub上下载镜像，设置国内的Docker Hub源能够提升下载速度，这里使用[中科大的地址](http://mirrors.ustc.edu.cn/help/dockerhub.html)，在配置文件`/etc/docker/daemon.json`中配置Hub地址：

```
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/”]
}
```

保存，重启Docker服务：

```shell
$ sudo service docker restart
```

验证是否生效：

```shell
$ sudo docker info
```

如果从结果中看到了如下内容，说明配置成功：

```
Registry Mirrors:
    https://docker.mirrors.ustc.edu.cn/
```

## Docker的基本用法

### 查询Docker信息

查询Docker信息，主要列出当前的容器数量和运行状态、镜像数量、Docker版本号、相关的资源和目录等等：

``` shell
$ sudo docker info
```

查询Docker版本号：

``` shell
$ sudo docker --verion
```

### 镜像操作

列出已下载的镜像：

``` shell
$ sudo docker images
```

删除名称为`hello-world`的镜像：

``` shell
$ sudo docker rmi hello-world
```

从远端仓库中搜索镜像，搜索Ubuntu镜像：

``` shell
$ sudo docker search ubuntu
```

这里选择排行第一的官方Ubuntu仓库的镜像进行下载：

``` shell
$ sudo docker pull ubuntu
```

存出镜像到文件，`ubuntu:latest`表示镜像仓库名（REPOSITORY）为`ubuntu`、标签名（TAG）为`latest`的镜像：

``` shell
$ sudo docker save -o /tmp/ubuntu.tar ubuntu:latest
```

载入文件到镜像：

``` shell
$ sudo docker load < /tmp/ubuntu.tar
```

### 容器操作

执行`docker run`命令可以指定从某个镜像创建一个容器，`-t` 选项表示创建虚拟终端，`-i`选项表示绑定标准输入，如果希望容器在后台运行，可以再加上`-d`选项：

``` shell
sudo docker run -it ubuntu
```

执行大概结果如下，自动进入Ubuntu容器：

```shell
root@b6bf45338f2d:/#
```

从容器中退出容器（容器会被关闭，因为退出了唯一的shell）：

``` shell
$ exit
```

列出已存在的容器，如不加`-a`选项则只列出运行中的容器：

```shell
$ sudo docker ps -a
```

通过以上命令可以看到容器ID、容器所属镜像、状态、名称等信息。Docker默认会自动生成容器名称，当然也可以在创建时使用`--name`选项指定容器名称：

``` shell
$ sudo docker run -it --name myubuntu ubuntu
```

容器的主机名称也可以指定：

``` shell
$ sudo docker run -it -h myhostname ubuntu
```

删除名称为`myubuntu`的容器：

```shell
$ sudo docker rm myubuntu
```

重启容器，指定容器ID，容器ID可以缩写，例如`b6bf45338f2d`可以缩写为`b6bf4`或`b6`甚至是`b`，只要缩写的ID是唯一的即可：

``` shell
$ sudo docker restart b6
```

也可以指定容器名称：

``` shell
$ sudo docker restart nervous_carson
```

使用`docker attach`进入容器：

``` shell
$ sudo docker attach b6
```

使用`docker exec`进入容器：

``` shell
$ sudo docker exec -it b6 /bin/bash
```

> attach和exec都可以进入容器，但是区别在于exec是执行命令的方式进入容器，所以如果指定执行/bin/bash，容器会新创建一个shell，这样的好处之一是执行exit退出容器时容器不会被关闭，而attach因为只有一个shell，退出后没有任何进程存在了，容器会被关闭。

终止容器：

``` shell
$ sudo docker stop b6
```

查看容器的详细信息：

``` shell
$ sudo docker inspect b6
```

导出容器：

``` shell
$ sudo docker export b6 > /tmp/mybuntu.tar
```

导入myubuntu.tar，创建一个名为myubuntu的镜像：

``` shell
$ sudo docker import - myubuntu < /tmp/myubuntu.tar
```

> 导出（export）导入（import）和存出（save）载入（load）都可以转储和迁移，但两者的区别在于：
>
> - export和import用于容器，会丢弃所有的历史记录和元数据信息，体积更小，相当于容器快照。
>
> - save和load用于镜像，会保存完整记录，体积更大，相当于镜像备份。
> - import可以重新指定镜像名称和标签，load不可以。

## 网络配置

### 端口映射

下载一个`nginx`镜像：

``` shell
$ sudo docker pull nginx
```

创建容器并启动，这里使用`-P`选项，docker随机映射一个本地（宿主机）的端口到nginx容器的80端口：

``` shell
$ sudo docker run -it -d -P nginx
```

执行`docker ps`可以看到`PORTS`列，本地的32768端口被映射到了容器的80端口：

```
PORTS
0.0.0.0:32768->80/tcp
```

这样访问宿主机的32768端口就等于是访问nginx容器的80端口，当然也可以指定宿主机要映射的端口，即使用`-p`选项，映射宿主机的80端口到nginx容器的80端口 ，格式为`宿主机端口:容器端口`：

``` shell
$ sudo docker run -it -d -p 80:80 nginx
```

### 容器互联

查看网络列表：

``` shell
$ sudo docker network ls
```

默认有3个网络，分别对应3种网络类型`bridge`、`host`和`null`：

```
NETWORK ID          NAME                DRIVER              SCOPE
5938cff7b158        bridge              bridge              local
1dd9839836f3        host                host                local
a5c66b5ba392        none                null                local
```

新建一个网络，类型为`bridge`，名称为`mynetwork`：

``` shell
$ sudo docker network create -d bridge mynetwork
```

下载`busybox`镜像，`busybox`集成了三百多个最常用的Linux命令和工具：

``` shell
$ sudo docker pull busybox
```

运行一个名称为`busybox1`的容器，并连接到网络`mynetwork`：

``` shell
$ sudo docker run -itd --name busybox1 --network mynetwork busybox
```

再运行一个名称为`busybox2`的容器，并连接到网络`mynetwork`：

``` shell
$ sudo docker run -itd --name busybox2 --network mynetwork busybox
```

为了验证`busybox1` 容器是否与`busybox2`建立了互联，分别进入两个容器，并尝试ping对方：

``` shell
$ sudo docker exec -it busybox1 sh
```

``` shell
/ # ping busybox2
```

```shell
$ sudo docker exec -it busybox2 sh
```

```shell
/ # ping busybox1
```

结果证明两个容器已经实现互联。

##数据管理

在容器中管理数据主要有两种方式：

- 卷（Volumes）
- 绑定挂载（Bind mounts）

### 卷

创建一个名称为`my-vol`的卷：

``` shell
$ sudo docker volume create my-vol
```

查看所有卷：

``` shell
$ sudo docker volume ls
```

查看卷的详细信息，可以看到卷在宿主机的挂载点：

``` shell
$ sudo docker volume inspect my-vol
```

删除卷：

``` shell
$ sudo docker volume rm my-vol
```

删除未使用的卷：

``` shell
$ sudo docker volume prune
```

启动一个容器，并使用`--mount`选项指定挂载一个卷`my-vol2`到容器的`/test`目录，启动容器时如果指定的卷不存在会自动创建：

``` shell
$ sudo docker run -itd --name ubuntu --mount source=my-vol2,target=/test ubuntu
```

这样在容器的/test目录下的数据，在宿主机或其他挂载了卷`my-vol2`的容器都可以共享和访问。重要的一点是即使容器被删除，卷里的数据仍然存在。

### 绑定挂载

启动一个容器，挂载主机目录`/tmp`到容器的`/test`目录，挂载类型为`bind`，挂载的主机目录必须已存在：

``` shell
$ sudo docker run -itd --name ubuntu2 --mount type=bind,source=/tmp,target=/test ubuntu
```

### 使用卷还是绑定挂载

两者都可以实现容器数据的持久化和共享，但是绑定挂载有明显的缺陷：

- 容器必须依赖于一个存在的主机目录，会影响容器可移植性。
- 在多个容器使用相同主机目录的情况下容易发生冲突。
- 容器可以直接使用、修改主机目录文件，存在安全隐患。

而卷算得上是对绑定挂载的改良，卷是由Docker管理，并与主机的核心区域隔离。

所以，应该尽可能的优先使用卷，而不是绑定挂载。

## 定制镜像

### 使用commit定制镜像

镜像是多层存储结构，而容器同样也是多层存储结构，容器是以镜像为基础层，加一层作为容器运行时的存储层。

我们在容器里做的任何文件修改都会被记录在容器的存储层里（卷除外），可以通过`docker diff`命令看到具体的改动。

当我们定制好了变化，我们希望能将其保存下来形成镜像，Docker提供了`docker commit`命令，可以将容器的存储层保存下来成为镜像，我们试着将名称为`ubuntu`的容器保存为名称为`myubuntu`、标签为`v1`的镜像：

``` shell
$ sudo docker commit ubuntu myubuntu:v1
```

使用`docker images` 就可以看到这个新定制的镜像。

可以查看这个镜像的历史记录：

``` shell
$ sudo docker history ubuntu-test:v1
```

`docker commit`虽然可以用来实现定制镜像，但鉴于以下几点原因，应该谨慎使用：

- 对镜像的操作都是黑箱操作，制作镜像时执行过什么命令，怎么生成的无从得知。
- 虽然`docker diff`可以得到一些线索，但远远不够。
- 因为是分层存储的结构，上一层的存储无法修改，在当前层删除上一层产生的文件，并不是真正被删除，而是被标记掉，这就导致每一次用commit制作镜像都会使镜像更加臃肿。

### 使用Dockerfile定制镜像

Dockerfile是一个文本文件的脚本，包含一条条的指令，一条指令构建一层，描述了该层应当如何构建。

Dockerfile解决了使用`docker commit`所带来的问题，下面说明如何编写和使用Dockerfile来定制镜像。

首先创建一个目录，新建一个Dockerfile文件：

``` shell
$ mkdir myubuntu
$ cd myubuntu
$ touch Dockerfile
```

编辑Dockerfile，内容为：

```
FROM ubuntu

RUN mkdir /testdir
RUN echo 'Dockerfile test' > /testdir/testfile
```

这个Dockerfile很简单，`FROM ubuntu`表示是以ubuntu镜像为基础进行定制

> 如果不想以任何镜像为基础，可以使用`FROM scratch`，scratch表示一个空白的镜像。

`RUN`指令是用来执行命令，这里我们创建了一个 `/testdir`目录，并在该目录下创建了一个`/tmp/test`文件。

Dockerfile中每一个指令都会建立一层，上面的Dockerfile有2个`RUN`指令，因此创建了2层，但由于分层存储是有层数限制的，所以应该减少不必要的层数，Dockerfile可以优化改写为：

```
FROM ubuntu

RUN mkdir /testdir \
    && echo 'Dockerfile test' > /testdir/testfile
```

然后就可以构建这个镜像了，在Dockerfile文件所在目录执行（请注意结尾有一个`.`，它的作用是将当前目录作为构建的上下文）：

``` shell
$ sudo docker build -t myubuntu:v2 .
```

使用`docker images` 就可以看到这个新定制的镜像。

除了`FROM`和`RUN`指令之外，docker还提供其他一些很有用的指令，比如`COPY`、`ADD`、`CMD`、`ENTRYPOINT`等等，这里暂不详细说明。

### 创建一个基础镜像

大多数情况下都可以在Docker Hub中找到我们需要的基础镜像，或者利用Dockerfile对基础镜像进行定制，但如果我们想创建一个全新的镜像，就需要费点功夫去打包一个Linux操作系统，具体如何打包取决于不同的Linux发行版，以Ubuntu举例，Ubuntu的打包比较简单。

我们准备一个虚拟机，已安装了Ubuntu，在这个虚拟机里先安装`debootstrap`：

``` shell
$ sudo apt install debootstrap
```

> debootstrap是debian/ubuntu下的一个工具，用来构建一套基本的系统(根文件系统)。

使用`debootstrap`构建一个基本的系统，命令格式大致如下：

``` shell
$ sudo debootstrap --arch=amd64 [版本代号] [下载目录] [指定软件源]
```

执行命令：

``` shell
$ sudo debootstrap --arch=amd64 bionic bionic http://mirrors.aliyun.com/ubuntu
```

执行后，`debootstrap`就会开始下载一个Ubuntu 18.04的基本系统到当前目录下的bionic目录，`bionic`是Ubuntu 18.04的版本代号。

可以chroot到这个基本系统，做一些操作，安装一些软件包之类的：

``` shell
$ sudo chroot bionic /bin/bash
```

操作完成清理系统并退出

``` shell
# rm -Rf /tmp/* && apt clean
```

``` shell
# exit
```

使用`tar`进行打包：

``` shell
$ sudo tar -cvf bionic.tar -C bionic .
```

最后使用`docker import`就可以导入成镜像：

``` shell
$ sudo cat bionic.tar | sudo docker import - [镜像名]
```

## 参考文档

- [https://docs.docker.com/install/linux/docker-ce/ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu)
- [http://mirrors.ustc.edu.cn/help/dockerhub.html](http://mirrors.ustc.edu.cn/help/dockerhub.html)
- [《Docker技术入门与实战》](https://book.douban.com/subject/26284823/)
- [https://yeasy.gitbooks.io/docker_practice/content/introduction/](https://yeasy.gitbooks.io/docker_practice/content/introduction/)