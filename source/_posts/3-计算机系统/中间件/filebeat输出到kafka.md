---
title: filebeat输出到kafka
tags:
  - kafka
  - filebeat
categories:
  - 3-计算机系统
  - 中间件
date: 2021-06-18 00:00:00
---

# 1. kafka

### 1.1 安装

+ kafka_2.13-2.8.0.tgz ,   前面的版本号是编译 Kafka 源代码的 Scala 编译器版本, 真正的版本号是2.8.0

```bash
wget https://mirrors.bfsu.edu.cn/apache/kafka/2.8.0/kafka_2.13-2.8.0.tgz
tar -xzf kafka_2.13-2.8.0.tgz 
cd kafka_2.13-2.8.0
```

<!-- more -->

+ 启动

```bash
# 一个终端
bin/zookeeper-server-start.sh config/zookeeper.properties

# 另外一个终端
bin/kafka-server-start.sh config/server.properties
```



+ 安装 java

  报错: /root/kafka_2.13-2.8.0/bin/kafka-run-class.sh: line 330: exec: java: not found
  Your local environment must have Java 8+ installed.

```bash
apt install openjdk-11-jre-headless
java --version
```



### 1.2 systemctl 服务

https://gist.github.com/vipmax/9ceeaa02932ba276fa810c923dbcbd4f

`vi /etc/systemd/system/kafka-zookeeper.service`

```ini
[Unit]
Description=Apache Zookeeper server (Kafka)
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target
[Service]
Type=simple
ExecStart=/data/tools/kafka-worth/bin/zookeeper-server-start.sh /data/tools/kafka-worth/config/zookeeper.properties
ExecStop=/data/tools/kafka-worth/bin/zookeeper-server-stop.sh
[Install]
WantedBy=multi-user.target
```



`vi /etc/systemd/system/kafka.service`

```ini
[Unit]
Description=Apache Kafka server (broker)
Documentation=http://kafka.apache.org/documentation.html
Requires=network.target remote-fs.target
After=network.target remote-fs.target kafka-zookeeper.service
[Service]
Type=simple
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
[Install]
WantedBy=multi-user.target
```

+ 启动

```bash
systemctl daemon-reload
systemctl start kafka-zookeeper.service
systemctl start kafka.service
```



### 1.3 kafka 使用

+ 创建一个 topic.

  ```bash
  bin/kafka-topics.sh --create --topic pingback --bootstrap-server localhost:9092
  
  # 删除topic
  bin/kafka-topics.sh --delete --topic pingback --bootstrap-server localhost:9092
  ```

+ 显示

  ```bash
  # 显示所有 topics
  bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
  
  
  # 显示特定 topic 属性
  bin/kafka-topics.sh --describe --topic pingback --bootstrap-server localhost:9092
  ```


+ 看组
  
  ```bash
  # 显示所有组
  bin/kafka-consumer-groups.sh  --list --bootstrap-server localhost:9092
  
  
  # 看特定组的消费情况
  bin/kafka-consumer-groups.sh --describe --group test-consumer-group --bootstrap-server localhost:9092
  ```
  
+ 生产
  
  ```bash
  bin/kafka-console-producer.sh --topic pingback --bootstrap-server localhost:9092
  #>This is my first event
  #>This is my second event
  #You can stop the producer client with Ctrl-C at any time.
  ```


+ 消费

  ```bash
  bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic pingback
  bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic pingback --from-beginning --group test-consumer-group
  
  #This is my first event
  #This is my second event
  #You can stop the consumer client with Ctrl-C at any time.
  ```
  
  
  
+ 重置offset

  ```bash
  # 回到最早该消费的点
  bin/kafka-consumer-groups.sh --group test-consumer-group --bootstrap-server localhost:9092 --reset-offsets --to-earliest --all-topics --execute
  
  # 重置到特定的 offset
  bin/kafka-consumer-groups.sh --group test-consumer-group --bootstrap-server localhost:9092 --reset-offsets --topic pingback:0 --to-offset 123 --execute
  ```

  提交过offset，latest和earliest没有区别，但是在没有提交offset情况下，用latest直接会导致无法读取旧数据。


+ 删除数据

  ```bash
  # 清理数据
  1. bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name pingback --alter --add-config retention.ms=1000
  2. bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name pingback --alter --delete-config retention.ms
  
  # 重置最初
  ```




### 1.4 配置

##### 1.4.1 kafka支持远程访问

打开config/server.properties配置文件，更改如下

把31行的注释去掉，listeners=PLAINTEXT://:9092
把36行的注释去掉，把advertised.listeners值改为PLAINTEXT://host_ip:9092



##### 1.4.2 修改 log 路径

Kafka的data目录是存储Kafka的数据文件的目录，是在${KAFKA_HOME}/config/server.properties中修改

```bash
log.dirs=/data/kafka_data
```

注意：log.dirs可以配置多个目录，需要用逗号分隔开



##### 1.4.3 周期删除数据

sudo vi config/server.properties

``` bash
log.retention.minutes=3
log.retention.hours=1  # 选一个
log.cleanup.policy=delete
```



# 2. filebeat

### 2.1 安装

```bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.1-linux-x86_64.tar.gz
tar zxvf filebeat-7.12.1-linux-x86_64.tar.gz
cd filebeat-7.12.1-linux-x86_64/
```



### 2.2 filebeat.yml

```ini
filebeat.inputs:
- type: log
  enabled: true
  paths:
      - /DATA/log/nginx/pb_ptio.access.log
  fields:
        document_type: online-pingback


output.kafka:
  hosts: ["localhost:9092"]
  topic: 'pingback'
  partition.round_robin:
    reachable_only: false
  compression: gzip


processors:
  - decode_json_fields:
      fields: ['message']
      target: ""
      overwrite_keys: false
      process_array: false
      max_depth: 1
  - decode_json_fields:
      fields: ['request_body']
      target: ""
      overwrite_keys: false
      process_array: false
      max_depth: 3
  - drop_fields:
      fields: ["message"]
```



### 2.3 systemctl 服务

`vi /etc/systemd/system/filebeat.service`

```ini
[Unit]
Description=filebeat
After=network.target
[Service]
Type=simple
Restart=yes
ExecStart=/opt/filebeat/filebeat -e -c /opt/filebeat/filebeat.yml
[Install]
WantedBy=multi-user.target
```

+ 启动

  ```bash
  systemctl start filebeat
  ```



# 3. golang 读取 kafka

```go
package main

import (
	"context"
	"flag"
	"log"
	"os"
	"os/signal"
	"strings"
	"sync"
	"syscall"

	"github.com/Shopify/sarama"
)

// Sarama configuration options
var (
	brokers  = ""
	version  = ""
	group    = ""
	topics   = ""
	assignor = ""
	oldest   = true
	verbose  = false
)

func init() {
	flag.StringVar(&brokers, "brokers", "", "Kafka bootstrap brokers to connect to, as a comma separated list")
	flag.StringVar(&group, "group", "", "Kafka consumer group definition")
	flag.StringVar(&version, "version", "2.1.1", "Kafka cluster version")
	flag.StringVar(&topics, "topics", "", "Kafka topics to be consumed, as a comma separated list")
	flag.StringVar(&assignor, "assignor", "range", "Consumer group partition assignment strategy (range, roundrobin, sticky)")
	flag.BoolVar(&oldest, "oldest", true, "Kafka consumer consume initial offset from oldest")
	flag.BoolVar(&verbose, "verbose", false, "Sarama logging")
	flag.Parse()

	if len(brokers) == 0 {
		panic("no Kafka bootstrap brokers defined, please set the -brokers flag")
	}

	if len(topics) == 0 {
		panic("no topics given to be consumed, please set the -topics flag")
	}

	if len(group) == 0 {
		panic("no Kafka consumer group defined, please set the -group flag")
	}
}

func main() {
	log.Println("Starting a new Sarama consumer")

	if verbose {
		sarama.Logger = log.New(os.Stdout, "[sarama] ", log.LstdFlags)
	}

	version, err := sarama.ParseKafkaVersion(version)
	if err != nil {
		log.Panicf("Error parsing Kafka version: %v", err)
	}

	/**
	 * Construct a new Sarama configuration.
	 * The Kafka cluster version has to be defined before the consumer/producer is initialized.
	 */
	config := sarama.NewConfig()
	config.Version = version

	switch assignor {
	case "sticky":
		config.Consumer.Group.Rebalance.Strategy = sarama.BalanceStrategySticky
	case "roundrobin":
		config.Consumer.Group.Rebalance.Strategy = sarama.BalanceStrategyRoundRobin
	case "range":
		config.Consumer.Group.Rebalance.Strategy = sarama.BalanceStrategyRange
	default:
		log.Panicf("Unrecognized consumer group partition assignor: %s", assignor)
	}

	if oldest {
		config.Consumer.Offsets.Initial = sarama.OffsetOldest
	}

	/**
	 * Setup a new Sarama consumer group
	 */
	consumer := Consumer{
		ready: make(chan bool),
	}

	ctx, cancel := context.WithCancel(context.Background())
	client, err := sarama.NewConsumerGroup(strings.Split(brokers, ","), group, config)
	if err != nil {
		log.Panicf("Error creating consumer group client: %v", err)
	}

	wg := &sync.WaitGroup{}
	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			// `Consume` should be called inside an infinite loop, when a
			// server-side rebalance happens, the consumer session will need to be
			// recreated to get the new claims
			if err := client.Consume(ctx, strings.Split(topics, ","), &consumer); err != nil {
				log.Panicf("Error from consumer: %v", err)
			}
			// check if context was cancelled, signaling that the consumer should stop
			if ctx.Err() != nil {
				return
			}
			consumer.ready = make(chan bool)
		}
	}()

	<-consumer.ready // Await till the consumer has been set up
	log.Println("Sarama consumer up and running!...")

	sigterm := make(chan os.Signal, 1)
	signal.Notify(sigterm, syscall.SIGINT, syscall.SIGTERM)
	select {
	case <-ctx.Done():
		log.Println("terminating: context cancelled")
	case <-sigterm:
		log.Println("terminating: via signal")
	}
	cancel()
	wg.Wait()
	if err = client.Close(); err != nil {
		log.Panicf("Error closing client: %v", err)
	}
}

// Consumer represents a Sarama consumer group consumer
type Consumer struct {
	ready chan bool
}

// Setup is run at the beginning of a new session, before ConsumeClaim
func (consumer *Consumer) Setup(sarama.ConsumerGroupSession) error {
	// Mark the consumer as ready
	close(consumer.ready)
	return nil
}

// Cleanup is run at the end of a session, once all ConsumeClaim goroutines have exited
func (consumer *Consumer) Cleanup(sarama.ConsumerGroupSession) error {
	return nil
}

// ConsumeClaim must start a consumer loop of ConsumerGroupClaim's Messages().
func (consumer *Consumer) ConsumeClaim(session sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
	// NOTE:
	// Do not move the code below to a goroutine.
	// The `ConsumeClaim` itself is called within a goroutine, see:
	// https://github.com/Shopify/sarama/blob/master/consumer_group.go#L27-L29
	for message := range claim.Messages() {
		log.Printf("Message claimed: value = %s, timestamp = %v, topic = %s", string(message.Value), message.Timestamp, message.Topic)
		session.MarkMessage(message, "")
	}

	return nil
}
```



# 4. 参考资料

+ https://www.liuvv.com/p/f03714cc.html
+ https://blog.csdn.net/u010889616/article/details/80640330
+ https://birdben.github.io/2016/12/15/Kafka/Kafka%E5%AD%A6%E4%B9%A0%EF%BC%88%E5%9B%9B%EF%BC%89%E6%8C%87%E5%AE%9AKafka%E7%9A%84data%E5%92%8Clogs%E8%B7%AF%E5%BE%84/
+ https://blog.csdn.net/qq_41926119/article/details/104510481
+ https://github.com/Shopify/sarama
