---
title: k8s的初探
tags:
  - k8s
categories:
  - 2-linux系统
  - k8s
abbrlink: '890e8359'
date: 2019-12-17 12:04:01
---

### 0. 前言

Kubernetes中的大部分概念Node、Pod、Replication Controller、Service等都可以看作一种“资源对象”，几乎所有的资源对象都可以通过kubectl工具（API调用）执行增、删、改、查等操作并将其保存在etcd中持久化存储。从这个角度来看，kubernetes其实是一个高度自动化的资源控制系统，通过跟踪对比etcd库里保存的“资源期望状态”与当前环境中的“实际资源状态”的差异来实现自动控制和自动纠错的高级功能。

<!-- more -->

+ Master：集群控制管理节点，所有的命令都经由master处理。

+ Node：是kubernetes集群的工作负载节点。Master为其分配工作，当某个Node宕机时，Master会将其工作负载自动转移到其他节点。

+ Pod：是kubernetes最重要也是最基本的概念。每个Pod都会包含一个 “根容器”，还会包含一个或者多个紧密相连的业务容器。

+ Label：是一个key=value的键值对，其中key与value由用户自己指定。可以附加到各种资源对象上，一个资源对象可以定义任意数量的Label。可以通过LabelSelector（标签选择器）查询和筛选资源对象。



### 1. Kubernetes 本地安装

我们需要安装以下东西：Kubernetes 的命令行客户端 kubctl、一个可以在本地跑起来的 Kubernetes 环境 Minikube。

#### 1.1 安装 kub

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF



yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
```



#### 1.2 安装 minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
   && sudo install minikube-linux-amd64 /usr/local/bin/minikube
   
   
minikube start --vm-driver=none --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```



为什么增加后面的image-repository, 因为GFW总是会在无形中增加学习的难度. 请参考: https://github.com/kubernetes/minikube/issues/3860



minikube 启动时会自动配置 kubectl，把它指向 Minikube 提供的 Kubernetes API 服务。可以用下面的命令确认：

```bash
$ kubectl config current-context
minikube
```



### 2. 使用

典型的 Kubernetes 集群包含一个 master 和多个 node。每个 node 上运行着维护 node 状态并和 master 通信的 kubelet。作为一个开发和测试的环境，Minikube 会建立一个有一个 node 的集群，用下面的命令可以看到：

```bash
$ kubectl get nodes
NAME       STATUS    AGE       VERSION
minikube   Ready     1h        v1.10.0
```



#### 2.1 创建 docker 容器

```bash
mkdir html
echo '<h1>Hello Kubernetes!</h1>' > html/index.html
```



`Dockerfile`

```dockerfile
FROM nginx
COPY html/* /usr/share/nginx/html
```

创建:

```bash
docker build -t k8s-demo:0.1 .
```



#### 2.2 创建pod

`pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: k8s-demo
    labels:
     app: k8s-demo
spec:
    containers:
        - name: k8s-demo
          image: k8s-demo:0.1
          ports:
              - containerPort: 80
```
创建:

```bash
kubectl create -f pod.yml
 

kubectl get pods
# NAME       READY     STATUS    RESTARTS   AGE
# k8s-demo   1/1       Running   0          5s


# 修改 pod.yml, 并应用
kubectl apply -f pod.yml
```



虽然这个 pod 在运行，我们无法从外部直接访问。要把服务暴露出来，我们需要创建一个 Service。Service 的作用有点像建立了一个反向代理和负载均衡器，负责把请求分发给后面的 pod。

#### 2.3 创建service

`svc.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
    name: k8s-demo-svc
    labels:
        app: k8s-demo
spec:
    type: NodePort
    ports:
        - port: 80
          nodePort: 30050
    selector:
        app: k8s-demo
```

这个 service 会把容器的 80 端口从 node 的 30050 端口暴露出来。

注意文件最后两行的 selector 部分，这里决定了请求会被发送给集群里的哪些 pod。这里的定义是所有包含「app: k8s-demo」这个标签的 pod。



查看标签命令:

```bash
kubectl describe pods | grep Labels
```



创建:

```bash
kubectl create -f svc.yml
```



用下面的命令可以得到暴露出来的 URL，在浏览器里访问，就能看到我们之前创建的网页了。

```bash
minikube service k8s-demo-svc --url
# http://10.0.0.5:30050

curl http://10.0.0.5:30050
# Hello Kubernetes!
```



#### 2.4 创建 deployment

在正式环境中我们需要让一个服务不受单个节点故障的影响，并且还要根据负载变化动态调整节点数量，所以不可能像上面一样逐个管理 pod。

Kubernetes 的用户通常是用 Deployment 来管理服务的。一个 deployment 可以创建指定数量的 pod 部署到各个 node 上，并可完成更新、回滚等操作。

`deployment.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-demo-deployment
spec:
  replicas: 10
  minReadySeconds: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: k8s-demo
    spec:
      containers:
        - name: k8s-demo-pod
          image: k8s-demo:0.1
          ports:
            - containerPort: 80
  selector:
    matchLabels:
      app: k8s-demo
```

创建:

```bash
kubectl create -f deployment.yml

# 用下面的命令可以看到这个 deployment 的副本集（replica set），有 10 个 pod 在运行。

kubectl get rs
# NAME                             DESIRED   CURRENT   READY     AGE
# k8s-demo-deployment-774878f86f   10        10        10        19s
```



假设我们对项目做了一些改动，要发布一个新版本。这里作为示例，我们只把 HTML 文件的内容改一下, 然后构建一个新版镜像 k8s-demo:0.2：

```bash
echo '<h1>Hello Kubernetes22222222222!</h1>' > html/index.html
docker build -t k8s-demo:0.2 .

# 替换 deployment.yml 的 tag 值

# 重新应用 deployment
kubectl apply -f deployment.yml --record=true
```



此时我们访问

```bash
curl http://10.0.0.5:30050
# Hello Kubernetes22222222222!
```



回滚版本1

``` bash
kubectl rollout undo deployment k8s-demo-deployment --to-revision=1

# 可以查看回滚进度
kubectl rollout status deployment k8s-demo-deployment
```



再次访问

```bash
curl http://10.0.0.5:30050
# Hello Kubernetes!
```



### 3. 总结

+ 容器(可以不是 docker) 放在 pod 里面
+ pod 增加了标签后, service 可以管理



### 4. 参考资料

+ https://zhuanlan.zhihu.com/p/39937913
