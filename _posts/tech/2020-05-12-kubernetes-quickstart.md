---
layout: post
title: Kubernetes快速上手
description: ""
category: 技术
tags: [ Kubernetes，K8s,PaaS ]
---

## 基本概念

### Cluster

Cluster 是计算、存储和网络资源的集合，Kubernetes 利用这些资源运行各种基于容器的应用。

### Master

Master 是 Cluster 的大脑，它的主要职责是调度，即决定将应用放在哪里运行。Master 运行 Linux 操作系统，可以是物理机或者虚拟机。为了实现高可用，可以运行多个 Master。

> 在集群管理方面，Kubernetes 将集群中的机器划分为一个 Master 节点和一群工作节点（Node）。其中，在 Master 节点上运行着集群管理相关的一组进程 kube-apiserver、kube-controller-manager 和 kube-scheduler，这些进程实现了整个集群的资源管理、Pod 调度、弹性伸缩、安全控制、系统监控和纠错等管理功能，并且都是全自动完成的。

### Node

Node 的职责是运行容器应用。Node 由 Master 管理，Node 负责监控并汇报容器的状态，同时根据 Master 的要求管理容器的生命周期。Node 运行在 Linux 操作系统上，可以是物理机或者是虚拟机。

> Node 作为集群中的工作节点，运行着真正的应用程序，在 Node 上 Kubernetes 管理的最小运行单元是 Pod。Node 上运行着 Kubernetes 的 kubelet、kube-proxy 服务进程，这些服务进程负责 Pod 的创建、启动、监控、重启、销毁，以及实现软件模式的负载均衡器。

### Pod

Pod 是 Kubernetes 的最小工作单元。每个 Pod 包含一个或多个容器。Pod 中的容器内给会作为一个整体被 Master 调度到一个 Node 上运行。

Kubernetes 引入 Pod 主要基于两个目的：

1. 可管理性：Pod 提供比容器更高层次的抽象，把一些紧密联系的容器封装到一个部署单元中。Kubernetes 以 Pod 为最小单位进行调度、扩展、共享资源、管理生命周期。
2. 通信和资源共享：Pod 中的所有容器使用同一个网络 namespace，即相同的 IP 地址和 Port 空间。它们可以直接用 localhost 通信。同样的，这些容器可以共享存储，当 Kubernetes 挂载 volume 到 Pod，本质上是将 volume 挂载到 Pod 中的每一个容器。

Pod 有两种使用方式：

1. 运行单一容器：只有一个容器，最常见的模型。
2. 运行多个容器：一般如果两个容器是紧密协作的，例如需要通过 volume 共享数据，则放到一个 Pod 是合适的；而例如像 Tomcat 和 MySQL 它们需要协作，但是通过 JDBC 交换数据，则没必要放到一个 Pod，放到各自的 Pod 更合适。

> 每个服务进程被包装到相应的Pod中，使其成为 Pod 中运行的一个容器（Container）。
>
> 可以给每个 Pod 贴上一个标签（Label），比如给运行 MySQL 的 Pod 贴上 name=mysql 标签，然后给对应的 Service 定义标签选择器（Label Selector），比如选择条件为 name=mysql，则该 Service 作用于所有包含 name=mysql 标签的 Pod 上，实现 Service 与 Pod 的关联。
>
> 每个 Pod 里运行着一个特殊的被称之为 Pause 的容器，其他容器则为业务容器，这些业务容器共享 Pause 容器的网络栈和 Volume 挂载卷，因此它们之间的通信和数据交换更为高效，可以利用这一特性将一组密切相关的服务进程放入同一个 Pod 中。
>
> Pod 运行在一个我们称之为节点（Node）的环境中，这个节点既可以是物理机，也可以说私有云或者公有云中的一个虚拟机，通常在一个节点上运行几百个 Pod。

### Controller

Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。Controller 中定义了 Pod 的部署特性，比如有几个副本、在什么样的 Node 上运行等。为了满足不同的业务场景，Kubernetes 提供了多种 Controller，包括 Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等。

- Deployment 是最常用的 Controller，可以通过创建 Deployment 来部署应用。Deployment 可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行。
- ReplicaSet 实现了 Pod 的多副本管理。使用 Deployment 时会自动创建 ReplicaSet，也就是说 Deployment 是通过 ReplicaSet 来管理 Pod 的多个副本的，我们通常不需要直接使用 ReplicaSet。
- DaemonSet 用于每个 Node 最多只运行一个 Pod 副本的场景。正如其名称所揭示的，DaemonSet 通常用于运行 daemon。
- StatefuleSet 能够保证 Pod 的每个副本在整个生命周期中名称是不变的，而其他 Controller 不提供这个功能。当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化，同时 StatefuleSet 会保证副本按照固定的顺序启动、更新或者删除。
- Job 用于运行结束就删除的应用，而其他 Controller 中的 Pod 通常是长期持续运行。

### Service

Kubernetes Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的 IP 和端口，Service 为 Pod 提供了负载均衡。

Kubernetes 运行容器（Pod）与访问容器（Pod）这两项任务分别由 Controller  和 Service 执行。

Service 提供的几种访问方式：

1. ClusterIP

   ClusterIP 是 Kubernetes 默认的服务类型，只有集群内的节点和 Pod 可以访问，不具备集群外部访问的能力。

2. NodePort

   NodePort 是外部访问服务的最基本方式，Kubernetes 会分配一个端口（默认范围300000 ~ 32767，不指定则随机分配），集群的每个节点都会代理这个端口。用户通过 `<NodeIP>:NodePort` 地址访问。

3. LoadBalancer

   Service 利用 cloud provider 特有的 load balancer 对外提供服务，cloud provider 负责将 load balancer 的流量导向 Service。目前支持的 cloud provider 有 GCP、AWS、Azur等。

4. Ingress

   Ingress 并不是服务类型的一种，它位于多个服务的前端，充当一个智能路由或者集群的入口点。

   GKE 默认的 Ingress 控制器将启动一个 HTTP(S) 的负载均衡器，这将使用户可以基于访问路径和子域名将流量路由到后端服务。例如，你可以将 `app.sample.com` 下的流量转发到 APP 服务， 将 `sample.com/app1/` 路径下的流量转发到 APP1 服务。

### Namespace

Namespace 可以将一个物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace。不同 Namespace 里的资源是完全隔离的。

Kubernetes 默认创建了两个 Namespace：

- default：创建资源时如果不指定，将被放到这个 Namespace 中。
- kube-system：Kubernetes 自己创建的系统资源将放到这个 Namespace 中。

## Kubernetes 核心原理

### Kubernetes API Server

核心功能是提供了 Kubernetes 各类资源对象（如 Pod、RC、Service 等）的增删改查及 Watch 等 HTTP Rest 接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。以及以下一些功能特性：

1. 是集群管理的 API 入口
2. 是资源配额控制的入口
3. 提供了完备的集群安全机制

Kubernetes API Server 通过一个名为 kube-apiserver 的进程提供服务，该进程运行在 Master 节点上。默认使用8080端口（对应参数 --insecure-port）提供 REST 服务。可以同时启动 HTTPS 安全端口（--secure-port=6443）来启动安全机制。

通常我们可以通过命令行工具 kubectl 来与 Kubernetes API Server 交互，它们之间的接口是 REST 调用。

如果只想对外暴露部分 REST 服务，则可以在 Master 或其他任何节点上通过运行 kubectl proxy 进程启动一个内部代理来实现。kubectl proxy 的特性包括可以设置拒绝客户端访问某些 API，或者采用白名单限制非法客户端访问等等。

## 部署 Kubernetes 集群

本例使用 kubeadm 部署 Kubernetes 集群，kubeadm 是用于快速构建 Kubernetes 集群的工具。

以部署最小的 Kubernetes 集群为例，准备2台主机，操作系统均为 Cent OS 7：

192.168.232.167 k8s-master

192.168.232.168 k8s-node1

**注意，以下步骤需要在各主机上同时操作。**

### 主机名称解析

编辑各主机的 `/etc/hosts` 文件，加入主机名称解析：

```
192.168.232.167 k8s-master
192.168.232.168 k8s-node1
```

### 关闭防火墙服务

``` shell
# systemctl stop firewalld iptables
# systemctl disable firewalld
# systemctl disable iptables
```

### 关闭并禁用 SELinux

修改 `/etc/selinux/config` 文件，将 `SELINUX=enforcing` 改为 `SELINUX=disabled`，然后重启主机。

### 安装 Docker

获取 docker-ce 的配置仓库配置文件：

``` shell
# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker.repo
```

安装 docker：

``` shell
# yum install docker-ce
```

另外，Docker 自1.13版起会自动设置 iptables 的 FORWARD 默认策略为 DROP，这可能会影响 Kubernetes 集群依赖的报文转发功能，因此，需要在 docker 服务启动后，重新将 FORWARD 链的默认策略设置为 ACCEPT，方式是修改 `/usr/lib/systemd/system/docker.service` 文件，在 `ExecStart=/usr/bin/dockerd` 一行之后新增一行如下内容：

```
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT
```

启动 docker 服务，并设置为随系统引导而自动启动：

``` shell
# systemctl daemon-reload
# systemctl start docker
# systemctl enable docker
```

Docker 需要从 Docker Hub 上下载镜像，设置国内的 Docker Hub 源能够提升下载速度，这里使用[中科大的地址](http://mirrors.ustc.edu.cn/help/dockerhub.html)，在配置文件 `/etc/docker/daemon.json` （如文件不存在则新建）中配置 Hub 地址：

```
{
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```

然后重启 Docker。

### 安装 kubelet 和 kubeadm

新建 kubernetes 的 yum 仓库配置文件 `/etc/yum.repos.d/kubernetes.repo`，使用了阿里云的源，内容如下：

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

安装 kubelet kubeadm kubectl：

``` shell
# yum install -y kubelet kubeadm kubectl
```

Kubernetes 自1.8版本起强制要求关闭系统上的交换分区（Swap），否则 kubelet 将无法启动，可以通过参数忽略此限制，编辑 kubelet 的配置文件 `/etc/sysconfig/kubelet`，设置如下参数：

```
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```

或者永久关闭Swap：

```shell
# swapoff -a
# sed -i '/swap/d' /etc/fstab
```

启动 kubelet 并设置开机自动启动：

``` shell
# systemctl enable kubelet && systemctl start kubelet
```

### 获取镜像列表

由于官方镜像地址被墙，所以我们需要先获取所需镜像以及它们的版本，然后从国内镜像站获取。

``` shell
# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.18.2
k8s.gcr.io/kube-controller-manager:v1.18.2
k8s.gcr.io/kube-scheduler:v1.18.2
k8s.gcr.io/kube-proxy:v1.18.2
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.2-0
k8s.gcr.io/coredns:1.6.7
```

然后通过执行下面的脚本从阿里云获取：

``` shell
images=(  # 下面的镜像应该去除"k8s.gcr.io/"的前缀，版本换成上面获取到的版本
    kube-apiserver:v1.18.2
    kube-controller-manager:v1.18.2
    kube-scheduler:v1.18.2
    kube-proxy:v1.18.2
    pause:3.2
    etcd:3.4.2-0
    coredns:1.6.7
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

### 集群初始化

**注意，从本步骤开始，只在指定节点主机上操作。**

在 Master 节点上执行初始化：

``` shell
# kubeadm init \
--kubernetes-version=v1.18.2 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12 \
--apiserver-advertise-address=0.0.0.0 \
--ignore-preflight-errors=Swap
```

```
参数说明：
--kubernetes-version：正在使用的 Kubernetes 程序组件的版本号，需要与 kubelet 的版本号相同。
--pod-network-cidr：Pod 网络的地址范围，其值为 CIDR 格式的网络地址；使用 flannel 网络插件时，其默认地址为 10.244.0.0/16
--service-cidr：Service 的网络地址范围，其值为 CIDR 格式的网络地址；默认地址为 10.96.0.0/12
--apiserver-advertise-address：API Server 通告给其他组件的 IP 地址，一般应该为 Master 节点的 IP 地址，0.0.0.0表示节点上所有可用的地址。
--ignore-preflight-errors：忽略哪些运行时的错误信息，其值为 Swap 时，表示忽略因 swap 未关闭而导致的错误。
```

初始化顺利完成后，会给出后续步骤的提示，如下：

```
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.232.167:6443 --token ifxtvp.1s2sa5gttcll3l75 --discovery-token-ca-cert-hash sha256:3998eae4e9c93632ec25413a11a3911d04545ea6bd979cf66282ec9568c19433
```

如果初始化失败，必要时可以使用 `kubeadm reset` 命令重置后重新初始化。

### 设置 kubectl 的配置文件

使用普通用户执行：

``` shell
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 验证集群状态

查询集群组件状态当前状态：

``` shell
$ kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

查询集群节点状态：

``` shell
$ kubectl get nodes
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   NotReady   master   19m   v1.18.2
```

可以看到 Master 节点的状态是 NotReady。

### 部署网络插件

为 Kubernetes 提供 Pod 网络的插件有很多，比较流行的有 flannel 和 Calico，这里选择 [flannel](https://github.com/coreos/flannel)：

``` shell
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

配置 flannel 网络插件时，Master 节点上的 Docker 首先会去获取 flannel 的镜像文件，而后根据镜像文件启动相应的 Pod 对象，需要等待一段时间，期间可以查看网络插件的状态：

``` shell
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                 READY   STATUS     RESTARTS   AGE
kube-system   coredns-86c58d9df4-fnxjm             0/1     Pending    0          35m
kube-system   coredns-86c58d9df4-vvjs9             0/1     Pending    0          35m
kube-system   etcd-k8s-master                      1/1     Running    0          35m
kube-system   kube-apiserver-k8s-master            1/1     Running    0          35m
kube-system   kube-controller-manager-k8s-master   1/1     Running    0          35m
kube-system   kube-flannel-ds-amd64-b5cdn          0/1     Init:0/1   0          10m
kube-system   kube-proxy-gdzzr                     1/1     Running    0          35m
kube-system   kube-scheduler-k8s-master            1/1     Running    0          35m
```

有的 Pod 不是 Running 状态，表示还没有就绪，可以查看指定 Pod 的具体情况：

``` shell
$ kubectl describe pod kube-flannel-ds-amd64-b5cdn --namespace=kube-system
```

如果网络质量不好，则需要耐心等待一段时间，Kubernetes 会重试。

一旦所有 Pod 都处于 Running，再次查看集群节点状态：

``` shell
$ kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   66m   v1.18.2
```

可以看到，此时的 Master 节点就处于 Ready 状态。

### 添加 Node 至集群中

根据集群初始化的最后提示以及生成的认证令牌（token），在 Node 节点上使用 root 用户执行：

``` shell
# kubeadm join 192.168.232.167:6443 --ignore-preflight-errors=Swap --token ifxtvp.1s2sa5gttcll3l75 --discovery-token-ca-cert-hash sha256:3998eae4e9c93632ec25413a11a3911d04545ea6bd979cf66282ec9568c19433
```

同样，为了忽略 Kubernetes 要求强制关闭 Swap 的限制，以上命令额外加上了 `--ignore-preflight-errors=Swap` 选项。

同样需要等待节点加入集群，之后可以在 Master 节点上查询一下节点状态：

``` shell
$ kubectl get nodes
NAME         STATUS   ROLES    AGE     VERSION
k8s-master   Ready    master   86m     v1.18.2
k8s-node1    Ready    <none>   8m19s   v1.18.2
```

至此，使用 kubeadm 部署 Kubernetes 集群完成。

### 允许 Pod 调度到 Master 节点（不推荐）

出于安全考虑，默认情况下 Master 节点被设置为"污点（Taints）"，即不允许 Pod 调度到 Master 节点上。

Taints 的取值有：

- NoSchedule：一定不能被调度
- PreferNoSchedule：尽量不要调度
- NoExecute：不仅不会调度，还会驱逐节点上已有的 Pod

通过查看节点信息可以看到 Taints 配置：

``` shell
$ kubectl describe node k8s-master

Name:               k8s-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=k8s-master
                    node-role.kubernetes.io/master=
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"b2:b4:a4:51:f2:59"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.232.167
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 07 Mar 2019 22:01:28 +0800
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
...
```

如果想让 Master 节点也成为工作节点，可以如下设置：

``` shell
$ kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule-
```

即删除 key 为 `k8s-master node-role.kubernetes.io`，值为 `NoSchedule` 的 Taints 配置。

### kubectl 命令补全

``` shell
$ sudo yum install -y bash-completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### 其他常用命令

查询集群状态：

``` shell
$ kubectl cluster-info
```

查询版本信息：

``` shell
$ kubectl cluster-info --short=true
```

查询 Pod 状态（显示 IP 和所在 Node 节点）：

``` shell
$ kubectl get pods --all-namespaces -o wide
```

## 部署应用

### 部署MySQL

新建一个 `mysql-5.7.yaml` 文件，内容如下：

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      protocol: TCP
      targetPort: 3306
      nodePort: 30001
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
        volumeMounts:
        - name: localtime
          mountPath: /etc/localtime
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
```

然后执行命令进行部署：

``` shell
$ kubectl apply -f mysql-5.7.yaml
```

Kubernetes 会在默认的 `Namespace`分别创建了一个 `Service` 和一个 `Deployment`，并且这个 `mysql ` 提供 `NodePort` 的方式对外提供访问，等待 Pod 启动运行后，我们便可以用节点的 IP 和 nodePort 访问这个 mysql（如 `192.168.232.167:30001`）。

上面创建的 mysql 虽然可以正常运行和工作，但是如果 Pod 重启后，之前的数据都会丢失。因此我们还需要给 MySql 指定一个持久化的存储。

### 持久化存储

要给 Pod 提供存储最简单的方式是使用卷（Volume），但是使用卷存在一些不足，一般需要和节点耦合，不优雅且未必能解决问题。

Kubernetes 对持久化存储提供了一个优雅的解决方案，即 `PersistentVolume`  和 `PersistentVolumeClaim`，简称 `PV` 和 `PVC`。通过绑定外部存储创建一个持久卷来支持数据的持久化，且生命周期是独立于 Pod ，解耦了存储的提供方和使用方。使用方只需要关心需要什么样的存储并进行挂载，而不需要关心存储的提供和实际位置。

Kubernetes 支持多种类型的 `PersistentVolume`  ，比如 `NFS`、`Ceph`、`GlusterFS` 等，这里以 `NFS` 举例。

### 安装 NFS 服务

由于资源有限，我们在 ` k8s-master` 节点上搭建一个 NFS 服务器，但合理的做法是专门提供一台 NFS 的服务器。

安装 NFS：

``` shell
$ sudo yum install nfs-utils
```

启动 NFS 服务：

``` shell
$ sudo systemctl enable nfs-server
$ sudo systemctl start nfs-server
```

创建 NFS 共享目录：

``` shell
$ sudo mkdir /srv/pv-1
$ sudo chown nfsnobody:nfsnobody /srv/pv-1
$ sudo chmod 755 /srv/pv-1
```

配置 NFS 共享目录，编辑 `/etc/exports`，写入以下内容：

``` 
/srv/pv-1  192.168.232.0/24(rw,sync,no_root_squash)
```

生效共享目录：

``` shell
$ sudo exportfs -a
```

### 创建 PersistentVolume

使用上面提供的 NFS 共享目录，创建一个 PV，容量为10Gi，新建一个 `pv-1.yaml` 文件：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /srv/pv-1
    server: k8s-master
```

执行命令进行创建：

``` shell
$ kubectl apply -f pv-1.yaml
```

完成后查看 PV 列表：

``` shell
$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          ...
pv-1   10Gi       RWO            Recycle          Bound    default/pvc-1  ...
```

### 创建 PersistentVolumeClaim

使用已创建的 PV 创建一个 PVC，即对存储的申请，申请5Gi，新建一个 `pvc-1.yaml`  文件：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

执行命令进行创建：

```shell
$ kubectl apply -f pvc-1.yaml
```

完成后查看 PVC 列表：

```shell
$ kubectl get pvc
NAME    STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-1   Bound    pv-1     10Gi       RWO                           1m
```

### 部署持久化的MySQL

接着我们就可以使用这个 PVC，并将其挂载到 MySQL 的数据文件目录 `/var/lib/mysql`，编辑 `mysql-5.7.yaml`：

``` yaml
apiVersion: v1                        
kind: Service                         
metadata:                             
  name: mysql                         
spec:                                 
  type: NodePort                      
  ports:                              
    - port: 3306                      
      protocol: TCP                   
      targetPort: 3306                
      nodePort: 30001                 
  selector:                           
    app: mysql                        
---                                   
apiVersion: apps/v1                   
kind: Deployment                      
metadata:                             
  name: mysql                         
spec:                                 
  replicas: 1                         
  selector:                           
    matchLabels:                      
      app: mysql                      
  template:                           
    metadata:                         
      labels:                         
        app: mysql                    
    spec:                             
      containers:                     
      - name: mysql                   
        image: mysql:5.7              
        imagePullPolicy: IfNotPresent 
        ports:                        
        - containerPort: 3306         
        env:                          
        - name: MYSQL_ROOT_PASSWORD   
          value: "root"               
        volumeMounts:                 
        - name: localtime             
          mountPath: /etc/localtime   
        - name: data-volume           
          mountPath: /var/lib/mysql   
      volumes:                        
      - name: localtime               
        hostPath:                     
          path: /etc/localtime        
      - name: data-volume             
        persistentVolumeClaim:        
          claimName: pvc-1
```

应用新的 yaml 文件：

``` shell
$ kubectl apply -f mysql-5.7.yaml
```

此时，MySQL 拥有了一个持久化的外部存储，即使 Pod 重启，数据仍然保留。

## 参考文档

[Kubernetes Documentation](https://kubernetes.io/docs/home/)

[《Kubernetes进阶实战》](https://book.douban.com/subject/30435129/)

[《每天5分钟玩转Kubernetes》](https://book.douban.com/subject/30186113/)