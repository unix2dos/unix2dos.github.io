---
title: nsq介绍和go-nsq使用
tags:
  - golang
  - nsq
abbrlink: 17c769c4
categories:
  - 1-编程语言
  - golang
date: 2018-05-19 11:01:13
---

### nsq 介绍

[nsq](https://github.com/bitly/nsq)是一个基于Go语言的分布式实时消息平台，nsq可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。



nsq具有分布式、去中心化的拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。NSQ非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。另外，官方还提供了拆箱即用Go和Python库。如果读者兴趣构建自己的客户端的话，还可以参考官方提供的[协议规范](http://nsq.io/clients/tcp_protocol_spec.html)。



nsq是由四个重要组件构成：

- [nsqd](http://bitly.github.io/nsq/components/nsqd.html)：一个负责接收、排队、转发消息到客户端的守护进程
- [nsqlookupd](http://bitly.github.io/nsq/components/nsqlookupd.html)：管理拓扑信息并提供最终一致性的发现服务的守护进程
- [nsqadmin](http://bitly.github.io/nsq/components/nsqadmin.html)：一套Web用户界面，可实时查看集群的统计数据和执行各种各样的管理任务
- [utilities](http://nsq.io/components/utilities.html)：常见基础功能、数据流处理工具，如nsq_stat、nsq_tail、nsq_to_file、nsq_to_http、nsq_to_nsq、to_nsq



nsq的主要特点如下:

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

<!-- more -->

### nsqlookupd(中心管理服务)  4160 4161
0. nsqlookupd是守护进程负责管理拓扑信息。客户端通过查询 nsqlookupd 来发现指定话题（topic）的生产者，并且 nsqd 节点广播话题（topic）和通道（channel）信息

1. 简单的说nsqlookupd就是中心管理服务，它使用tcp(默认端口4160)管理nsqd服务，使用http(默认端口4161)管理nsqadmin服务。同时为客户端提供查询功能

2. nsqlookupd具有以下功能或特性

* 唯一性，在一个Nsq服务中只有一个nsqlookupd服务。当然也可以在集群中部署多个nsqlookupd，但它们之间是没有关联的
* 去中心化，即使nsqlookupd崩溃，也会不影响正在运行的nsqd服务
* 充当nsqd和naqadmin信息交互的中间件
* 提供一个http查询服务，给客户端定时更新nsqd的地址目录 



### nsqadmin(展示数据)

0. 是一套 WEB UI，用来汇集集群的实时统计，并执行不同的管理任务
1. nsqadmin具有以下功能或特性
    * 提供一个对topic和channel统一管理的操作界面以及各种实时监控数据的展示，界面设计的很简洁，操作也很简单
    * 展示所有message的数量，恩....装X利器
    * 能够在后台创建topic和channel，这个应该不常用到
    * nsqadmin的所有功能都必须依赖于nsqlookupd，nsqadmin只是向nsqlookupd传递用户操作并展示来自nsqlookupd的数据



### nsqd (真正干活的)  4150 4151

1. nsqd 是一个守护进程，负责接收，排队，投递消息给客户端
2. 简单的说，真正干活的就是这个服务，它主要负责message的收发，队列的维护。nsqd会默认监听一个tcp端口(4150)和一个http端口(4151)以及一个可选的https端口
3. nsqd 具有以下功能或特性

* 对订阅了同一个topic，同一个channel的消费者使用负载均衡策略（不是轮询）
* 只要channel存在，即使没有该channel的消费者，也会将生产者的message缓存到队列中（注意消息的过期处理）
* 保证队列中的message至少会被消费一次，即使nsqd退出，也会将队列中的消息暂存磁盘上(结束进程等意外情况除外)
* 限定内存占用，能够配置nsqd中每个channel队列在内存中缓存的message数量，一旦超出，message将被缓存到磁盘中
* topic，channel一旦建立，将会一直存在，要及时在管理台或者用代码清除无效的topic和channel，避免资源的浪费

#### 消费者

消费者有两种方式与nsqd建立连接

* 消费者直连nsqd，这是最简单的方式，缺点是nsqd服务无法实现动态伸缩了(当然，自己去实现一个也是可以的)  
* 消费者通过http查询nsqlookupd获取该nsqlookupd上所有nsqd的连接地址，然后再分别和这些nsqd建立连接(官方推荐的做法)，但是客户端会不停的向nsqlookupd查询最新的nsqd地址目录(不喜欢用http轮询这种方式...)



#### 生产者

生产者必须直连nsqd去投递message(网上说，可以连接到nsqlookupd，让nsqlookupd自动选择一个nsqd去完成投递，但是我用Producer的tcp是连不上nsqlookupd的，不知道http可不可以...)，

这里有一个问题就是如果生产者所连接的nsqd炸了，那么message就会投递失败，所以在客户端必须自己实现相应的备用方案



* Producer断线后不会重连，需要自己手动重连，Consumer断线后会自动重连
* Consumer的重连时间配置项有两个功能(这个设计必须吐槽一下，分开配置更好一点)

    * Consumer检测到与nsqd的连接断开后，每隔x秒向nsqd请求重连
    * Consumer每隔x秒，向nsqlookud进行http轮询，用来更新自己的nsqd地址目录
    * Consumer的重连时间默认是60s(...菜都凉了)，我改成了1s
* Consumer可以同时接收不同nsqd node的同名topic数据，为了避免混淆，就必须在客户端进行处理
* 在AddConurrentHandlers和 AddHandler中设置的接口回调是在另外的goroutine中执行的
* Producer不能发布(Publish)空message，否则会导致panic



### nsq 操作

```
brew install nsq


nsqlookupd

[nsqlookupd] 2018/01/24 10:33:31.710309 nsqlookupd v1.0.0-compat (built w/go1.9.1)
[nsqlookupd] 2018/01/24 10:33:31.710772 TCP:
listening on [::]:4160  --nsqd
[nsqlookupd] 2018/01/24 10:33:31.710772 HTTP: 
listening on [::]:4161 --nsqadmin


nsqd --lookupd-tcp-address=127.0.0.1:4160 -broadcast-address=127.0.0.1

[nsqd] 2018/01/24 10:35:01.975420 nsqd v1.0.0-compat (built w/go1.9.1)
[nsqd] 2018/01/24 10:35:01.975521 ID: 660
[nsqd] 2018/01/24 10:35:01.975567 NSQ: persisting topic/channel metadata to nsqd.dat
[nsqd] 2018/01/24 10:35:01.976544 TCP: 
listening on [::]:4150  --tcp监听4150
[nsqd] 2018/01/24 10:35:01.976597 HTTP: 
listening on [::]:4151  --http监听4151
[nsqd] 2018/01/24 10:35:01.976785 LOOKUP(127.0.0.1:4160): adding peer


nsqadmin --lookupd-http-address=127.0.0.1:4161

[nsqadmin] 2018/01/24 10:35:40.562980 nsqadmin v1.0.0-compat (built w/go1.9.1)
[nsqadmin] 2018/01/24 10:35:40.563388 HTTP: 
listening on [::]:4171 --监听4171
```

#### use

可以用浏览器访问 http://127.0.0.1:4171/ 观察数据
也可尝试下 watch -n 0.5 "curl -s http://127.0.0.1:4151/stats" 监控统计数据(4151是nsqd的http)


发布一个消息 
curl -d 'hello world 1' 'http://127.0.0.1:4151/pub?topic=test'

创建一个消费者:
nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161


再发布几个消息

```
curl -d 'hello world 2' 'http://127.0.0.1:4151/pub?topic=test'
curl -d 'hello world 3' 'http://127.0.0.1:4151/pub?topic=test'
```



### go-nsq 使用

运行之前, 保证 nsqd 在运行

```
package main

import (
	"log"
	"time"

	nsq "github.com/bitly/go-nsq"
)

func main() {
	go startConsumer()
	startProducer()
}

// 生产者
func startProducer() {
	cfg := nsq.NewConfig()
	producer, err := nsq.NewProducer("127.0.0.1:4150", cfg)
	if err != nil {
		log.Fatal(err)
	}
	// 发布消息
	for {
		if err := producer.Publish("test", []byte("test message")); err != nil {
			log.Fatal("publish error: " + err.Error())
		}
		time.Sleep(1 * time.Second)
	}
}

// 消费者
func startConsumer() {
	cfg := nsq.NewConfig()
	consumer, err := nsq.NewConsumer("test", "sensor01", cfg)
	if err != nil {
		log.Fatal(err)
	}
	// 设置消息处理函数
	consumer.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
		log.Println(string(message.Body))
		return nil
	}))
	// 连接到单例nsqd
	if err := consumer.ConnectToNSQD("127.0.0.1:4150"); err != nil {
		log.Fatal(err)
	}
	<-consumer.StopChan
}

```

