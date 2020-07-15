---
title: zookeeper介绍和使用
tags:
  - zookeeper
categories:
  - 3-计算机系统
  - 中间件
abbrlink: 7c519e18
date: 2019-04-07 00:00:00
---


# 1. 分布式应用

### 1.1 一致性

强一致性

弱一致性

最终一致性

<!-- more -->

### 1.2 选举Leader

先看 zid (写的一个日志 id),  再看 myid

选取 Leader 时间节点

+ 服务刚启动

+ leader 挂了

+ 超过一半的节点挂了， leader shutdown

### 1.3 同步操作

leader 负责写

follower 转发给 leader 写

+ leader 生成日志，发给所有 follower

+ follewer 持久化日志， 回复 ack

+ leader 接收超过一半 ack, 发送commit,  更新 database

+ follewer收到 commit, 更新 database



# 2. zookeeper

### 2.1 znode

+ path 唯一路径

+ data 数据, 可以没有

+ childNode 子节点

+ stat 状态属性

+ type 节点类型

  + 持久节点（除非手动删除，节点永远存在, 默认的）
  + 持久有序节点（按照创建顺序会为每个节点末尾带上一个序号如：root-1）
  + 临时节点（创建客户端与 Zookeeper 保持连接时节点存在，断开时则删除并会有相应的通知）
    + 不能拥有子节点
    + 服务注册
    + 分布式锁
  + 临时有序节点（在瞬时节点的基础上加上了顺序）

  

# 3. 安装和使用

### 3.1 安装

```bash
brew install zookeeper
```

安装完成，配置文件在 `/usr/local/etc/zookeeper/` 

### 3.2 使用

```bash
zkServer start #启动 zookeeper
zkCli # 连接 zookeeper

ls /
create /apps "test"
get -s /apps

create -s # 序号
create -e # 临时
```



# 4. 分布式锁
通过临时和序号的原理
+ 创建临时序号节点
+ 获取最小的节点是否时读锁, 如果是读, 那么获得锁
+ 如果不是, 阻塞等待, 添加子节点变更监听(可以改进链表, 监听它的上一个节点)



# 5. 服务注册和发现

### 5.1 服务注册

服务提供者（Provider）启动时，会向Zookeeper服务端注册服务信息，即会在Zookeeper服务器上创建一个服务节点(临时节点)，并在节点上存储服务的相关数据（如服务提供者的ip地址、端口等），比如注册一个用户注册服务（user/register）。

### 5.2 服务发现

服务消费者（Consumer）启动时，会根据本身依赖的服务信息，向Zookeeper服务端获取注册的服务信息并设置Watch，获取到注册的服务信息之后将服务提供者信息缓存在本地，调用服务时直接根据从Zookeeper注册中心获取到的服务注册信息调用服务，比如发现用户注册服务（user/register）并调用。

### 5.3 服务通知

当服务提供者因为某种原因宕机或不提供服务之后，Zookeeper服务注册中心的对应服务节点会被删除，因为消费者在获取服务信息的时候在对应节点上设置了Watch，因此节点删除之后会触发对应的Watcher，Zookeeper注册中心会异步向服务所关联的所有服务消费者发出节点删除的通知，服务消费者根据收到的通知更新缓存的服务列表。

### 5.4 总结

+ 注册使用临时节点, 保存 ip 和端口, 就算挂了节点也能释放
+ 消费者监听父节点, 服务变换就会收到通知
+ 根据列表中的一系列服务, 可以随机算法实现调用, 相当于负载均衡