---
title: kafka介绍和使用
tags:
  - golang
  - kafka
categories:
  - 3-计算机系统
  - 中间件
abbrlink: f03714cc
date: 2019-06-07 00:00:00
---

# 1. kafka介绍


Kafka 是 linkedin 用于日志处理的分布式消息队列，同时支持离线和在线日志处理。kafka 对消息保存时根据 Topic 进行归类，发送消息者成为 Producer,消息接受者成为 Consumer,此外 kafka 集群有多个 kafka 实例组成，每个实例(server)称为 broker。无论是 kafka集群，还是 producer 和 consumer 都依赖于 zookeeper 来保证系统可用性，为集群保存一些 meta 信息。

<!-- more -->

### 1.1 redis 消息订阅和发布的区别

老板有个好消息要告诉大家，有两个办法:
1.到每个座位上挨个儿告诉每个人。什么？张三去上厕所了？那张三就只能错过好消息了！
2.老板把消息写到黑板报上，谁想知道就来看一下，什么？张三请假了？没关系，我一周之后才擦掉，总会看见的！什么张三请假两周？那就算了，我反正只保留一周，不然其他好消息没地方写了

redis用第一种办法，kafka用第二种办法，知道什么区别了吧



# 2. 安装和使用

### 2.1 mac

```bash
brew install kafka  # 如果报错，提示安装 java, 安装即可

zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties  # 临时启动，先启动 zookeeper, 再启动 kafaka

# 此时用 ps 查看进程可以看到是用 java 起来的。
```

### 2.3 使用

创建Topic

```bash
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

产生消息

```bash
kafka-console-producer --broker-list localhost:9092 --topic test
```

消费

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning
```

如果使用消费组

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic test --group test-consumer1 --from-beginning
```



# 3.  golang 使用 kafka

github 上有多个轮子,选用了star最多的 sarama

+ https://github.com/Shopify/sarama 
+ https://github.com/confluentinc/confluent-kafka-go

### 3.1 生产者

```go
package main

import (
	"fmt"

	"github.com/Shopify/sarama"
)

func main() {
	config := sarama.NewConfig()
	config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
	config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
	config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

	// 连接kafka
	client, err := sarama.NewSyncProducer([]string{"127.0.0.1:9092"}, config)
	if err != nil {
		fmt.Println("producer closed, err:", err)
		return
	}
	defer client.Close()
	
	// 发送消息
	msg := &sarama.ProducerMessage{}
	msg.Topic = "web_log"
	msg.Value = sarama.StringEncoder("this is a test log1")
	pid, offset, err := client.SendMessage(msg)
	if err != nil {
		fmt.Println("send msg failed, err:", err)
		return
	}
	fmt.Printf("pid:%v offset:%v\n", pid, offset)
}


```

### 3.2 消费者

```go
package main

import (
	"fmt"

	"github.com/Shopify/sarama"
)

func main() {
	consumer, err := sarama.NewConsumer([]string{"localhost:9092"}, nil)
	if err != nil {
		fmt.Printf("fail to start consumer, err:%v\n", err)
		return
	}

	// 取出老的值
	oldest, _ := consumer.ConsumePartition("web_log", 0, sarama.OffsetOldest)
	defer oldest.AsyncClose()
	for msg := range oldest.Messages() {
		fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v\n", msg.Partition, msg.Offset, string(msg.Key), string(msg.Value))
	}
	
	// 消费新增加的值
	newest, _ := consumer.ConsumePartition("web_log", 0, sarama.OffsetNewest)
	defer newest.AsyncClose()
	for msg := range newest.Messages() {
		fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v\n", msg.Partition, msg.Offset, string(msg.Key), string(msg.Value))
	}
}
```




# 4. 参考资料

+ https://colobu.com/2019/09/27/install-Kafka-on-Mac/
