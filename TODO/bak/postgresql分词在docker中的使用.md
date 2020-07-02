---
title: postgresql分词在docker中的使用
tags:
  - postgresql
  - docker
categories:
  - 1-编程语言
  - sql
abbrlink: 660ad2f0
date: 2019-11-14 21:11:46
---



### 1. 分词安装

PostgreSQL默认没有中文分词功能，因此要实现PostgreSQL的中文分词功能必须使用扩展插件。常用的是两种中文分词插件，pg_jieba和Zhparser。pg_jieba是基于结巴中文分词的，zhparser是基于SCWS。

<!-- more -->

##### 1.1 安装zhparser

+ scws 安装

  ```bash
  wget -q -O - http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2 | tar xf -
  cd scws-1.2.3 
  ./configure 
  make install 
  ```

+ zhparse安装

  ```bash
  git clone https://github.com/amutu/zhparser.git 
  cd zhparser
  PG_CONFIG=/usr/lib/postgresql/9.5/bin/pg_config make && make install
  ```

  

### 2. docker postgresql

```bash
docker run -e DB_NAME=dbname -e DB_USER=user -e DB_PASS=mypassword -e TZ=Hongkong -p 54323:5432 -d -v  /Users/liuwei/data/:/var/lib/postgresql --name name1 lcgc/postgresql:9.5.4

# 如果不行, 请用下面的, 参见 https://stackoverflow.com/a/41650891
-v  /Users/liuwei/data:/var/lib/postgresql/data


# 其他容器复用
docker run -e DB_NAME=dbname -e DB_USER=user -e DB_PASS=mypassword -e TZ=Hongkong -p 54323:5432 -d -v  /Users/liuwei/data:/var/lib/postgresql --name name2 lcgc/postgresql:9.5.4
```



```bash
\l  #显示数据库

\d  #显示表

\d table  #显示表结构

# 如果查询需要在宿主机显示输出, 参见 https://stackoverflow.com/a/41892369
docker exec -it kinema-db1 psql -U kinema -d kinema -c "SELECT * FROM ks_cinema"
```



docker分词仓库

https://github.com/ChiChou/zhparser-docker



### 3. 参考资料

+ https://www.jianshu.com/p/0baedb2c550d

+ https://www.cnblogs.com/zhenbianshu/p/7795247.html

+ https://github.com/ChiChou/zhparser-docker