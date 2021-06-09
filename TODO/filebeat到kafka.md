# 1. kafka

### 1.0 介绍

1. kafka 消息一直保留, 但是有有效期
2. kafka 有顺序的, 只是针对一个 part
3. kafka_2.13-2.8.0.tgz ,   前面的版本号是编译 Kafka 源代码的 Scala 编译器版本, 真正的版本号是2.8.0

### 1.1 安装

```bash
$ wget https://mirrors.bfsu.edu.cn/apache/kafka/2.8.0/kafka_2.13-2.8.0.tgz
$ tar -xzf kafka_2.13-2.8.0.tgz 
$ cd kafka_2.13-2.8.0
```

启动

```bash
# 一个终端
bin/zookeeper-server-start.sh config/zookeeper.properties

# 另外一个终端
bin/kafka-server-start.sh config/server.properties
```



报错: /root/kafka_2.13-2.8.0/bin/kafka-run-class.sh: line 330: exec: java: not found
Your local environment must have Java 8+ installed.

安装 java

```bash
apt install openjdk-11-jre-headless
java --version
```



### 1.2 systemctl 配置

https://gist.github.com/vipmax/9ceeaa02932ba276fa810c923dbcbd4f

vi /etc/systemd/system/kafka-zookeeper.service

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



vi /etc/systemd/system/kafka.service

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



```bash
systemctl daemon-reload
systemctl start kafka-zookeeper.service
systemctl start kafka.service
```



### 1.3 配置

##### 1.3.1 支持远程访问

打开config/server.properties配置文件，更改如下

把31行的注释去掉，listeners=PLAINTEXT://:9092
把36行的注释去掉，把advertised.listeners值改为PLAINTEXT://host_ip:9092（我的服务器ip是192.1683.45）

https://blog.csdn.net/u010889616/article/details/80640330

##### 1.3.2 log 配置

https://birdben.github.io/2016/12/15/Kafka/Kafka%E5%AD%A6%E4%B9%A0%EF%BC%88%E5%9B%9B%EF%BC%89%E6%8C%87%E5%AE%9AKafka%E7%9A%84data%E5%92%8Clogs%E8%B7%AF%E5%BE%84/

##### 1.3.3 周期删除数据

sudo vi config/server.properties

``` bash
log.retention.minutes=3
log.retention.hours=1  # 选一个
log.cleanup.policy=delete
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

  





# 2. filebeat 安装

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



https://blog.csdn.net/qq_41926119/article/details/104510481

### 2.3 服务

vi /etc/systemd/system/filebeat.service

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

systemctl start filebeat





# 3. golang 读取 kafka

https://zhuanlan.zhihu.com/p/31731892

