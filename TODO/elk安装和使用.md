---
title: "elk安装和使用"
date: 2021-01-20 00:00:01
tags:
- elasticsearch
- elk
- linux
---

# 1. 安装

### 1.1 安装elasticsearch

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz
tar -xzf elasticsearch-7.10.1-linux-x86_64.tar.gz
cd elasticsearch-7.10.1/bin
./elasticsearch
```

Make sure Elasticsearch is up and running```curl http://127.0.0.1:9200```

<!-- more -->

+ 为了安全不允许 root 启动

  ```bash
  adduser elasticsearch
  chown -R elasticsearch:elasticsearch elasticsearch-7.10.1
  su elasticsearch
  ```

  

+ could not find java in bundled jdk at

  ```bash
  apt install default-jre
  ```

  设置环境变量

  ```bash
  vi /etc/default/elasticsearch
  JAVA_HOME=/usr
  START_DAEMON=true
  ES_USER=elasticsearch
  ES_GROUP=elasticsearch
  
  export JAVA_HOME=/usr
  ```

  

### 1.2 安装kibana

```bash
wget https://mirrors.huaweicloud.com/kibana/7.10.1/kibana-7.10.1-linux-x86_64.tar.gz
tar xzvf kibana-7.10.1-linux-x86_64.tar.gz
cd kibana-7.10.1-linux-x86_64/
./bin/kibana
```



+ 访问外网 ip:5601不行

  ```bash
  vi config/kibana.yml
  #修改
  server.host: "0.0.0.0"
  ```

  

### 1.3 安装filebeat

```bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.10.1-linux-x86_64.tar.gz
tar -zxvf filebeat-7.10.1-linux-x86_64.tar.gz
cd filebeat-7.10.1-linux-x86_64
```



收集日志

```bash
sudo mkdir -p /opt/data/my-app1-log
```



编辑`/opt/data/my-app1-log/app.log`并存入以下测试示例日志内容：

```
2020-10-06 11:40:36.652 INFO http-bio-10009-exec-302 - start someMethod for some params
2020-10-06 11:40:36.652 INFO http-bio-10009-exec-302 - getUser for someField null,paramId 1111
2020-10-06 11:40:36.653 DEBUG http-bio-10009-exec-302 - ooo Using Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@5f324b3c]
2020-10-06 11:40:36.653 DEBUG http-bio-10009-exec-302 - ==>  Preparing: select * from t_user where login_name=?
2020-10-06 11:40:36.653 DEBUG http-bio-10009-exec-302 - ==> Parameters: 18888888888(String)
2020-10-06 11:40:37.254 INFO http-bio-10009-exec-302 - Filtering response of path: /some/controller/path
2020-10-06 11:40:37.254 INFO http-bio-10009-exec-308 - Filtering request of path: /some/controller/path
2020-10-06 11:40:37.254 INFO http-bio-10009-exec-308 - init session with something=<null>,xxxxxxxx,userId=1234
2020-10-06 11:40:37.254 INFO http-bio-10009-exec-308 - start someMethod for some params
2020-10-06 11:40:37.255 INFO http-bio-10009-exec-308 - getUser for someField null,paramId 2222
2020-10-06 11:40:37.255 DEBUG http-bio-10009-exec-308 - ooo Using Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@5f324b3c]
2020-10-06 11:40:37.255 DEBUG http-bio-10009-exec-308 - ==>  Preparing: select * from t_user where login_name=?
2020-10-06 11:40:37.255 DEBUG http-bio-10009-exec-308 - ==> Parameters: 19999999999(String)
2020-10-06 11:40:37.262 INFO http-bio-10009-exec-171 - Filtering request of path: /some/controller/path
2020-10-06 11:40:37.262 INFO http-bio-10009-exec-171 - init session with something=<null>,xxxxxxxx,userId=1256
2020-10-06 11:40:37.262 INFO http-bio-10009-exec-171 - Filtering response of path: /some/controller/path
2020-10-06 11:40:37.262 INFO http-bio-10009-exec-308 - Filtering response of path: /another/controller/path
2020-10-06 11:40:37.263 INFO http-bio-10009-exec-135 - Filtering request of path: /another/controller/path
2020-10-06 11:40:37.263 INFO http-bio-10009-exec-135 - init session with something=<null>,xxxxxxxx,userId=5678
2020-10-06 11:40:37.277 DEBUG http-bio-10009-exec-135 - ooo Using Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@5f324b3c]
2020-10-06 11:40:37.277 DEBUG http-bio-10009-exec-135 - ==>  Preparing: select a.* from sometable a where a.user_id = ?
2020-10-06 11:40:37.277 DEBUG http-bio-10009-exec-135 - ==> Parameters: 5678(Long)
2020-10-06 11:40:37.277 INFO http-bio-10009-exec-135 - Filtering response of path: /another/controller/path
```



配置输入源`vi filebeat.yml`

```ini
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /opt/data/my-app1-log/*.log
```



启动

```bash
./filebeat test config -e # 测试配置, 如果没有问题的话最后会输出Config OK字样。

./filebeat -e # 启动
```



### 1.4 安装 journalbeat

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/journalbeat/journalbeat-7.10.2-linux-x86_64.tar.gz
tar xzvf journalbeat-7.10.2-linux-x86_64.tar.gz


sudo chown root journalbeat.yml 
sudo ./journalbeat -e
```



journalbeat.yml

```bash
include_matches:
	- _SYSTEMD_UNIT=lw-music.service
```





# 2. 展示数据

### 2.1 创建索引模式

打开Kibana页面，点击菜单 Management > Stack Management > Index Patterns，然后点击页面上的Create index pattern，在Index pattern输入框中输入filebeat-*关键字，当提示Success! Your index pattern matches 1 index.时，我们点击Next step。



然后在Time Filter field name下拉列表中选择@timestamp作为时间过滤字段，最后点击Create index pattern按钮，稍等几秒，完成索引模式创建。

### 2.2 查询

点击菜单 Kibana > Discover, 进入查询页面，当前页面默认查询过去15分钟的日志，如果你的页面显示没有查到任何东西，请调整一下时间段，比如将时间改为查找今天，然后就能查到内容了.

![image-20210120170134220](elk%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/image-20210120170134220.png) 



# 3. 创建服务

### 3.1 elasticsearch

sudo vi /lib/systemd/system/elasticsearch.service

```bash
[Unit]
Description=elasticsearch
After=network.target
[Service]
Type=simple
User=elasticsearch
Group=elasticsearch
Restart=no
ExecStart=/home/elasticsearch/workspace/elasticsearch-7.10.1/bin/elasticsearch
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

systemctl start elasticsearch

### 3.2 kibana

sudo vi /lib/systemd/system/kibana.service

```
[Unit]
Description=kibana
After=network.target
[Service]
Type=simple
User=elasticsearch
Group=elasticsearch
Restart=no
ExecStart=/home/elasticsearch/workspace/kibana-7.10.1-linux-x86_64/bin/kibana
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

systemctl start kibana



### 3.3 journalbeat

sudo vi /lib/systemd/system/journalbeat.service

```
[Unit]
Description=journalbeat
After=network.target
[Service]
Type=simple
User=root
Group=root
Restart=no
WorkingDirectory=/home/elasticsearch/workspace/journalbeat-7.10.2-linux-x86_64/
ExecStart=/home/elasticsearch/workspace/journalbeat-7.10.2-linux-x86_64/journalbeat
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

systemctl start journalbeat





# 4. 参考资料

+ https://blog.csdn.net/cloud_xy/article/details/103228810
+ https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html
+ https://www.elastic.co/guide/cn/elasticsearch/guide/current/getting-started.html