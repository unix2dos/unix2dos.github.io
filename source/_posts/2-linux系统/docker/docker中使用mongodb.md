---
title: docker中使用mongodb
tags:
  - docker
  - mongodb
categories:
  - 2-linux系统
  - docker
abbrlink: 6fa8633a
date: 2019-10-19 12:57:46
---

### 1. 安装

##### 1.1 安装 mongodb

```bash
mkdir ~/data

sudo docker pull mongo:latest 

# 一定要把数据卷暴露出去, 这样方便数据迁移
sudo docker run -d -p 27017:27017 --name mongo -v /home/liuwei/data:/data/db mongo:latest

sudo docker exec -it mongo mongo
```

<!-- more -->



##### 1.2 将数据迁移到新容器

Let's start a new MongoDB container, this time running on port 37017 instead of the default 27017:

```bash
# Copy the data from the previous container 
sudo cp -r ~/data ~/data_clone  

# Start another MongoDB container 
sudo docker run -d -p 37017:27017 -v ~/data_clone:/data/db mongo
```



### 2. 使用

```sql
db.createCollection('cities') 
db.cities.insert({ name: 'New York', country: 'USA' }) 
db.cities.insert({ name: 'Paris', country: 'France' }) 
db.cities.find()
```



### 3. 参考资料

+ https://www.thachmai.info/2015/04/30/running-mongodb-container/

+ docker-compose 使用mongodb 参考 {% post_link 2-linux系统/docker/docker-compose的一次实践 %}

