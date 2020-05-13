---
layout: post
title: 在Kubernetes中部署Harbor
description: ""
category: 技术
tags: [ Harbor,Docker,Kubernetes,K8s ]
---

## 关于 Harbor

Harbor 是一个开源的企业级私有 Docker 镜像仓库，我们可以利用 Harbor 在本地安全地存储和管理 Docker 镜像，而不需要将镜像上传到 Docker Hub 或其他第三方镜像仓库。

## 安装 Helm

Helm 是 Kubernetes 的包管理器。就像是 Debian、Ubuntu 的 apt，或是 Red Hat、CentOS 的 yum。

### Helm 客户端

我们将 Helm 客户端安装在 k8s-master 节点上：

``` shell
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

安装完成后查看 Helm 的版本号：

``` shell
$ helm version
version.BuildInfo{Version:"v3.2.1", GitCommit:"fe51cd1e31e6a202cba7dead9552a6d418ded79a", GitTreeState:"clean", GoVersion:"go1.13.10"}
```

安装命令补全：

``` shell
$ echo 'source <(helm completion bash)' >> ~/.bashrc
$ source ~/.bashrc
```

### 修改 Helm 仓库镜像

添加阿里云镜像仓库

``` shell
$ helm repo add aliyun  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
$ helm repo update
```

查看仓库列表：

``` shell
$ helm repo list
```

## 安装 Harbor

添加 Harbor 仓库：

```shell
$ helm repo add harbor https://helm.goharbor.io
$ helm repo update
```

搜索 Harbor：

``` shell
$ helm search repo harbor/harbor

NAME            CHART VERSION   APP VERSION     DESCRIPTION
harbor/harbor   1.3.2           1.10.2           An open source...
```

先将 Harbor 下载到本地：

``` shell
$ helm fetch harbor/harbor
```

解压缩：

``` shell
$ tar xvf harbor-1.3.2.tgz
$ cd harbor
```

修改配置文件 `values.yaml`，具体查看GitHub上面的配置列表[Configuration](https://github.com/goharbor/harbor-helm/blob/master/README.md#configuration)。

这里修改了以下几个配置：

```yaml
expose:
  type: nodePort
  tls:
    enabled: false
  nodePort:
    ports:
      http:
        nodePort: 30004
      https:
        nodePort: 30005
      notary:
        nodePort: 30006
externalURL: http://192.168.232.167:30004
persistence:
  persistentVolumeClaim:
    registry:
      existingClaim: "pvc-harbor"
      storageClass: "-"
      subPath: "registry"
    chartmuseum:
      existingClaim: "pvc-harbor"
      storageClass: "-"
      subPath: "chartmuseum"
    jobservice:
      existingClaim: "pvc-harbor"
      storageClass: "-"
      subPath: "jobservice"
    database:
      existingClaim: "pvc-harbor"
      storageClass: "-"
      subPath: "database"
    redis:
      existingClaim: "pvc-harbor"
      storageClass: "-"
      subPath: "redis"
```

其中 `pvc-harbor` 是预先创建好的 PVC，创建过程略。

部署 Harbor：

``` shell
$ helm install -g harbor/harbor -f values.yaml .

NAME: harbor-1589352564
LAST DEPLOYED: Wed May 13 14:49:25 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Please wait for several minutes for Harbor deployment to complete.
Then you should be able to visit the Harbor portal at http://192.168.232.167:30004.
For more details, please visit https://github.com/goharbor/harbor.
```

部署成功，等待 Harbor 启动完成，然后就可以通过`http://192.168.232.167:30004` 访问Harbor，默认用户名是admin，密码是Harbor12345。

由于 Docker 自从 1.3.x 之后，docker registry 交互默认使用的是HTTPS，而我们搭建的 Harbor 使用的是HTTP，所以为了避免 pull/push 镜像时得到错误：`http: server gave HTTP response to HTTPS client`，需要修改 docker 的配置文件 `/etc/docker/daemon.json`，加入以下配置：

``` json
{
    "insecure-registries": ["192.168.232.167:30004"]
}
```

保存后重启 docker 服务：

``` shell
$ sudo systemctl restart docker
```

## 上传第一个镜像

下载一个 busybox 镜像：

``` shell
$ docker pull busybox:latest
```

修改 tag：

``` shell
$ docker tag busybox:latest 192.168.232.167:30004/library/busybox:latest
```

library 是 Harbor 的默认项目地址，也可以登录 Harbor 自行新建一个项目。

使用 docker login 登录到 Harbor，并输入用户名和密码：

```shell
$ docker login 192.168.232.167:30004
```

登录成功后，上传镜像到 Harbor：

``` shell
$ docker push 192.168.232.167:30004/library/busybox:latest
```

## 参考文档

[https://helm.sh/docs/using_helm/#installing-helm](https://helm.sh/docs/using_helm/#installing-helm)

[https://github.com/goharbor/harbor-helm](https://github.com/goharbor/harbor-helm)

[https://www.cnblogs.com/dukuan/p/9963744.html](https://www.cnblogs.com/dukuan/p/9963744.html)

[https://blog.51cto.com/wangpengtai/2418636?source=dra](https://blog.51cto.com/wangpengtai/2418636?source=dra)