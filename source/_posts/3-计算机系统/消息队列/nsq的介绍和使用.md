---
title: nsq的介绍和使用
tags:
  - golang
  - nsq
abbrlink: 17c769c4
categories:
  - 3-计算机系统
  - 消息队列
date: 2019-06-18 11:01:13
---

# 1. nsq 介绍

[nsq](https://github.com/bitly/nsq)是一个基于Go语言的分布式实时消息平台，nsq可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。

<!-- more -->

### 1.1 nsq 的组成

nsq是由四个重要组件构成：

- [nsqd](http://bitly.github.io/nsq/components/nsqd.html)：一个负责接收、排队、转发消息到客户端的守护进程
- [nsqlookupd](http://bitly.github.io/nsq/components/nsqlookupd.html)：管理拓扑信息并提供最终一致性的发现服务的守护进程
- [nsqadmin](http://bitly.github.io/nsq/components/nsqadmin.html)：一套Web用户界面，可实时查看集群的统计数据和执行各种各样的管理任务
- [utilities](http://nsq.io/components/utilities.html)：常见基础功能、数据流处理工具，如nsq_stat、nsq_tail、nsq_to_file、nsq_to_http、nsq_to_nsq、to_nsq

### 1.2 nsq的主要特点

- 具有分布式且无单点故障的拓扑结构 支持水平扩展，在无中断情况下能够无缝地添加集群节点
- 低延迟的消息推送，参见官方提供的[性能说明文档](http://nsq.io/overview/performance.html)
- 具有组合式的负载均衡和多播形式的消息路由
- 既擅长处理面向流（高吞吐量）的工作负载，也擅长处理面向Job的（低吞吐量）工作负载
- 消息数据既可以存储于内存中，也可以存储在磁盘中
- 实现了生产者、消费者自动发现和消费者自动连接生产者，参见nsqlookupd
- 支持安全传输层协议（TLS），从而确保了消息传递的安全性
- 具有与数据格式无关的消息结构，支持JSON、Protocol Buffers、MsgPacek等消息格式
- 非常易于部署（几乎没有依赖）和配置（所有参数都可以通过命令行进行配置）
- 使用了简单的TCP协议且具有多种语言的客户端功能库
- 具有用于信息统计、管理员操作和实现生产者等的HTTP接口
- 为实时检测集成了统计数据收集器[StatsD](https://github.com/etsy/statsd/)
- 具有强大的集群管理界面，参见nsqadmin



# 2. nsq 组件

### 2.1 nsqd (真正干活的)

1. nsqd 是一个守护进程，负责接收，排队，投递消息给客户端
2. 简单的说，真正干活的就是这个服务，它主要负责message的收发，队列的维护。nsqd会默认监听一个tcp端口(4150)和一个http端口(4151)以及一个可选的https端口
3. nsqd 具有以下功能或特性

* 对订阅了同一个topic，同一个channel的消费者使用负载均衡策略（不是轮询）
* 只要channel存在，即使没有该channel的消费者，也会将生产者的message缓存到队列中（注意消息的过期处理）
* 保证队列中的message至少会被消费一次，即使nsqd退出，也会将队列中的消息暂存磁盘上(结束进程等意外情况除外)
* 限定内存占用，能够配置nsqd中每个channel队列在内存中缓存的message数量，一旦超出，message将被缓存到磁盘中
* topic，channel一旦建立，将会一直存在，要及时在管理台或者用代码清除无效的topic和channel，避免资源的浪费



### 2.2 nsqlookupd(中心管理服务)

0. nsqlookupd是守护进程负责管理拓扑信息。客户端通过查询 nsqlookupd 来发现指定话题（topic）的生产者，并且 nsqd 节点广播话题（topic）和通道（channel）信息

1. 简单的说nsqlookupd就是中心管理服务，它使用tcp(默认端口4160)管理nsqd服务，使用http(默认端口4161)管理nsqadmin服务。同时为客户端提供查询功能

2. nsqlookupd具有以下功能或特性

* 唯一性，在一个Nsq服务中只有一个nsqlookupd服务。当然也可以在集群中部署多个nsqlookupd，但它们之间是没有关联的
* 去中心化，即使nsqlookupd崩溃，也会不影响正在运行的nsqd服务
* 充当nsqd和naqadmin信息交互的中间件
* 提供一个http查询服务，给客户端定时更新nsqd的地址目录 



### 2.3 nsqadmin(展示数据)

0. 是一套 WEB UI，用来汇集集群的实时统计，并执行不同的管理任务
1. nsqadmin具有以下功能或特性
    * 提供一个对topic和channel统一管理的操作界面以及各种实时监控数据的展示，界面设计的很简洁，操作也很简单
    * 展示所有message的数量，恩....装X利器
    * 能够在后台创建topic和channel，这个应该不常用到
    * nsqadmin的所有功能都必须依赖于nsqlookupd，nsqadmin只是向nsqlookupd传递用户操作并展示来自nsqlookupd的数据



# 3. nsq 操作

### 3.1 安装和启动

安装 nsq

``` bash
brew install nsq 
```

启动 nsqlookupd

```bash
nsqlookupd

[nsqlookupd] 2020/07/14 13:10:26.005144 INFO: nsqlookupd v1.2.0 (built w/go1.13.5)
[nsqlookupd] 2020/07/14 13:10:26.006620 INFO: HTTP: listening on [::]:4161 # 管理 nsqd
[nsqlookupd] 2020/07/14 13:10:26.006620 INFO: TCP: listening on [::]:4160 # 管理 nsqadmin
```

启动 nsqd

```bash
nsqd --lookupd-tcp-address=127.0.0.1:4160 -broadcast-address=127.0.0.1

[nsqd] 2020/07/14 13:11:55.889343 INFO: nsqd v1.2.0 (built w/go1.13.5)
[nsqd] 2020/07/14 13:11:55.889519 INFO: ID: 24
[nsqd] 2020/07/14 13:11:55.889954 INFO: NSQ: persisting topic/channel metadata to nsqd.dat
[nsqd] 2020/07/14 13:11:55.909569 INFO: TCP: listening on [::]:4150 # tcp监听4150
[nsqd] 2020/07/14 13:11:55.909673 INFO: HTTP: listening on [::]:4151 # http监听4151
[nsqd] 2020/07/14 13:11:55.909858 INFO: LOOKUP(127.0.0.1:4160): adding peer
[nsqd] 2020/07/14 13:11:55.909872 INFO: LOOKUP connecting to 127.0.0.1:4160
[nsqd] 2020/07/14 13:11:55.914186 INFO: LOOKUPD(127.0.0.1:4160): peer info {TCPPort:4160 HTTPPort:4161 Version:1.2.0 BroadcastAddress:liuweideMacBook-Air.local}
```

启动 nsqadmin

```bash
nsqadmin --lookupd-http-address=127.0.0.1:4161

[nsqadmin] 2020/07/14 13:13:39.580161 INFO: nsqadmin v1.2.0 (built w/go1.13.5)
[nsqadmin] 2020/07/14 13:13:39.581124 INFO: HTTP: listening on [::]:4171 # 监听4171
```

### 3.2 操作

+ 浏览器访问 http://127.0.0.1:4171/ 观察数据

+ 发布一个消息 

  ```bash
  curl -d 'levonfly1' 'http://127.0.0.1:4151/pub?topic=test'
  ```

+ 创建一个消费者

  ```bash
  nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161
  ```
  可以在/tmp/文件夹内看到输入的消息log

  

+ 再发布几个消息

  ```bash
  curl -d 'levonfly2' 'http://127.0.0.1:4151/pub?topic=test'
  curl -d 'levonfly3' 'http://127.0.0.1:4151/pub?topic=test'
  ```



# 4. golang 使用 nsq

### 4.1 生产者

```go
package main

import (
	"fmt"
	"log"
	"time"

	nsq "github.com/nsqio/go-nsq"
)

func main() {
	cfg := nsq.NewConfig()
	// 连接 nsqd 的 tcp 连接
	producer, err := nsq.NewProducer("127.0.0.1:4150", cfg)
	if err != nil {
		log.Fatal(err)
	}

	// 发布消息
	var count int
	for {
		count++
		body := fmt.Sprintf("test %d", count)
		if err := producer.Publish("test", []byte(body)); err != nil {
			log.Fatal("publish error: " + err.Error())
		}
		time.Sleep(1 * time.Second)
	}
}
```

### 4.2 消费者

```go
package main

import (
	"log"

	"github.com/nsqio/go-nsq"
)

func main() {
	cfg := nsq.NewConfig()
	consumer, err := nsq.NewConsumer("test", "levonfly", cfg)
	if err != nil {
		log.Fatal(err)
	}

	// 处理信息
	consumer.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
		log.Println(string(message.Body))
		return nil
	}))

	// 连接 nsqd 的 tcp 连接
	if err := consumer.ConnectToNSQD("127.0.0.1:4150"); err != nil {
		log.Fatal(err)
	}
	<-consumer.StopChan
}
```