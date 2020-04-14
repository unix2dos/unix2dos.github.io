---
title: k8s的原理和实践
tags:
  - k8s
  - linux
categories:
  - 2-linux系统
  - k8s
abbrlink: 40c9599
date: 2020-04-24 17:50:00
---



# 0. 前言

先来看一张 k8s 架构设计以及组件之间的通信协议

![1](k8s实践/2.jpeg)



+ Master 负责管理整个集群。Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。

+ Node 是一个虚拟机或者物理机，它在 Kubernetes 集群中充当工作机器的角色。
+ 在 Kubernetes 上部署应用时，您告诉 Master 启动应用容器。 Master 就编排容器在群集的 Node 上运行。 Node 使用 Master 暴露的 Kubernetes API 与 Master 通信。

<!-- more -->


下面是更抽象的一个视图：
![1](k8s实践/3.png)



Master架构:
![1](k8s实践/4.png)



Node架构:
![1](k8s实践/5.png)

最后走一遍 k8s 官网的交互式教程, 能够快速的对 k8s 有一个简单的了解.



# 1. kubectl 

+ kubectl version 

  client  version 是kubectl的版本

  server version 是 k8s 的版本

  

+ kubectl get nodes

  查看所有的 node 节点

  

+ kubectl get pods 

  查看 pod 节点

  

+ kubectl describe pods 

  查看 pod 状态, 例如 ip , 里面的容器之类的

  

+ kubectl describe services/kubernetes-bootcamp  

  查看services状态

  

+ kubectl get services 

  查看services



+ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080  

  新建 service 并暴露端口

  

+ kubectl delete service -l run=kubernetes-bootcamp  

  删除一个 service



+ kubectl create deployment  创建deployment

  kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

  

+ kubectl get deployments

  获取deployments



+ kubectl describe deployment 

  查看deployment

  

+ kubectl label pod $POD_NAME app=v1  

  给 pod 加个 lable

  

+ kubectl get pods -l app=v1  

  加完 lable 就能查看了

  

+ kubectl get rs 

  查看拷贝列表

  

+ kubectl scale deployments/kubernetes-bootcamp --replicas=2  

  直接复制品,牛逼了

  

+ kubectl logs  查看日志

  kubectl logs $POD_NAME

  

+ kubectl exec 执行命令

  kubectl exec $POD_NAME env  查看环境变量

  kubectl exec -ti $POD_NAME bash  进入到 pod 里面

  

+ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10

  更换 image

  

+ kubectl rollout status deployments/kubernetes-bootcamp 

  查看状态

  

+ kubectl rollout undo deployments/kubernetes-bootcamp  

  回滚





# 2. 问题记录

+ minikube是干啥的?

   minikube 是一种轻量级的 Kubernetes 实现，可在本地计算机上创建 VM 并部署仅包含一个节点的简单集群。

  简单理解为一个运行在本地Node，我们可以在里面创建Pods来创建对应的服务.



+ deployment和 pods的关系

  pod是单一亦或一组容器的合集, deployment是pod版本管理的工具 用来区分不同版本的pod, 可以回滚版本

  单独创建pod的时候就不会有deployment出现，但是创建deployment的时候一定会创建pod,因为pod是一个基础的单位。





# 3. 参考资料

+ https://kubernetes.io/zh/docs/tutorials/hello-minikube/
+ https://www.bookstack.cn/read/kubernetes-handbook/concepts-index.md

