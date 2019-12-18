---
title: "k8s的初探"
date: 2019-10-19 21:11:46
tags:
- k8s
---

### 0. 前言

Kubernetes中的大部分概念Node、Pod、Replication Controller、Service等都可以看作一种“资源对象”，几乎所有的资源对象都可以通过kubectl工具（API调用）执行增、删、改、查等操作并将其保存在etcd中持久化存储。从这个角度来看，kubernetes其实是一个高度自动化的资源控制系统，通过跟踪对比etcd库里保存的“资源期望状态”与当前环境中的“实际资源状态”的差异来实现自动控制和自动纠错的高级功能。

<!-- more -->

Master：集群控制管理节点，所有的命令都经由master处理。

Node：是kubernetes集群的工作负载节点。Master为其分配工作，当某个Node宕机时，Master会将其工作负载自动转移到其他节点。

Pod：是kubernetes最重要也是最基本的概念。每个Pod都会包含一个 “根容器”，还会包含一个或者多个紧密相连的业务容器。

Label：是一个key=value的键值对，其中key与value由用户自己指定。可以附加到各种资源对象上，一个资源对象可以定义任意数量的Label。可以通过LabelSelector（标签选择器）查询和筛选资源对象。



### 1. 安装

我们需要安装以下东西：Kubernetes 的命令行客户端 kubctl、一个可以在本地跑起来的 Kubernetes 环境 Minikube、以及给 Minikube 使用的虚拟化引擎 hyperkit。

```bash
brew install kubectl
brew cask install minikube
```



Minikube 默认的虚拟化引擎是 `VirtualBox`, 我们用`hyperkit` 进行替代

```bash
rm -rf ~/.minikube
minikube start --vm-driver=hyperkit --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

为什么增加后面的image-repository, 因为GFW总是会在无形中增加学习的难度. 请参考: https://github.com/kubernetes/minikube/issues/3860



如果你在第一次启动 Minikube 时遇到错误或被中断，后面重试仍然失败时，可以尝试运行 `minikube delete` 把集群删除，重新来过。过程如果比较慢, 请耐心等待.



Minikube 启动时会自动配置 kubectl，把它指向 Minikube 提供的 Kubernetes API 服务。可以用下面的命令确认：

```bash
$ kubectl config current-context
minikube
```



### 2. 使用

##### 2.1 Kubernetes 架构简介

典型的 Kubernetes 集群包含一个 master 和多个 node。Master 是控制集群的中心，node 是提供 CPU、内存和存储资源的节点。Master 上运行着很多进程，包括面向用户的 API 服务、负责维护集群状态的 Controller Manager、负责调度任务的 Scheduler 等。每个 node 上运行着维护 node 状态并和 master 通信的 kubelet，以及实现集群网络服务的 kube-proxy。

作为一个开发和测试的环境，Minikube 会建立一个有一个 node 的集群，用下面的命令可以看到：

```bash
$ kubectl get nodes
NAME       STATUS    AGE       VERSION
minikube   Ready     1h        v1.10.0
```



##### 2.2 部署一个单实例服务

Kubernetes 中部署的最小单位是 pod，而不是 Docker 容器。

实时上 Kubernetes 是不依赖于 Docker 的，完全可以使用其他的容器引擎在 Kubernetes 管理的集群中替代 Docker。在与 Docker 结合使用时，一个 pod 中可以包含一个或多个 Docker 容器。但除了有紧密耦合的情况下，通常一个 pod 中只有一个容器，这样方便不同的服务各自独立地扩展。

Minikube 自带了 Docker 引擎，所以我们需要重新配置客户端，让 docker 命令行与 Minikube 中的 Docker 进程通讯：

```bash
eval $(minikube docker-env) 
eval $(minikube docker-env -u) 
```

在运行上面的命令后，再运行 `docker image ls` 时只能看到一些 Minikube 自带的镜像.







### 3. 遇到的问题

##### 3.1 下载不下来

```bash
rm -rf ~/.minikube
minikube start --vm-driver=hyperkit --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```



##### 3.2 Get https://registry-1.docker.io/v2/: proxyconnect tcp: dial tcp 127.0.0.1:7890: connect: connection refused

把vpn代理去掉

```bash
rm -rf ~/.minikube
minikube start --vm-driver=hyperkit --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```



##### 3.3 Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on 192.168.64.1:53: no such host

Setting the DNS Server for the Docker Daemon via "Preferences > Daemon > Advanced" based upon advice given [here](https://github.com/moby/moby/issues/32270#issuecomment-290829987) by [@thaJeztah](https://github.com/thaJeztah) via putting in the following value worked for me (no reinstall required):

```
{ "dns" : [ "8.8.8.8", "8.8.4.4" ]}
```



##### 3.4 Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on 192.168.64.1:53: server misbehaving

https://github.com/kubernetes/minikube/issues/3036

https://github.com/kubernetes/minikube/issues/4594





### 4. 参考资料

+ https://zhuanlan.zhihu.com/p/39937913
+ https://blog.csdn.net/qq_35254726/article/details/54233781



### 