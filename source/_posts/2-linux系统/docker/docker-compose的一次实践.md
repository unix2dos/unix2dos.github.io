---
title: docker-compose的一次实践
tags:
  - docker
  - docker-compose
categories:
  - 2-linux系统
  - docker
abbrlink: 331c471e
date: 2019-10-19 12:11:46
---



### 0. 前言

Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用，它是由 `python` 编写。

`Compose` 定位是定义和运行多个 Docker 容器的应用。`Compose` 有两个重点

- `docker-compose.yml` `compose` 配置文件
- `docker-compose` 命令行工具

<!-- more -->

### 1. 安装

windows 和 mac 中 `docker-compose` 在安装 `docker` 的时候就已经捆绑安装了。linux 中需要自己安装


```bash
# 版本可以去 github 查看最新的版本
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-(uname -s)-(uname -m)" -o /usr/local/bin/docker-compose 


sudo chmod +x /usr/local/bin/docker-compose 

docker-compose --version
```



### 2. 使用

```bash
docker-compose up # 启动
docker-compose down # 关闭
```

> docker-compose.yml

```yml
version: '3' # 定义版本，不指定默认为版本 1，新版本功能更多


services:

  mongo4:
    image: mongo:4
    privileged: true
    restart: unless-stopped
    volumes:  
      - $HOME/transcode/data/db/:/data/db/
      - ./mongo/mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    container_name: mongo4
    environment:
      MONGO_INITDB_ROOT_USERNAME: yx1
      MONGO_INITDB_ROOT_PASSWORD: test
      MONGO_INITDB_DATABASE: transcode_v1
    network_mode: bridge # 加上不会创建默认的桥, 即有一个为空, 就会创建一个默认 network
    ports: # 暴露端口信息
      - "47047:27017"



  transcode-service:
    build: service # 指定 Dockerfile 所在文件夹的路径
    privileged: true # 允许容器中运行一些特权命令
    restart: unless-stopped
    volumes:
     - /var/lib/oceans/:/var/lib/oceans/
    container_name: transcode-service
    network_mode: host


  transcode-webapi:
    build: webapi
    privileged: true
    restart: unless-stopped
    container_name: transcode-webapi
    network_mode: host
```



然后在 webapi, service 文件夹内创建各自的 dockerfile 文件




> ./mongo/mongo-init.js

```javascript
db.createUser(
    {
        user: "yx1",
        pwd: "test",
        roles:[
            {
                role: "readWrite",
                db:   "transcode_v1"
            }
        ]
    }
);
```



##### 2.1 默认网桥问题

docker-compose 启动后会自动创建一个网桥, 如果不想创建, 即每个容器都写上值, 不能为空

```yaml
network_mode: bridge
```

参考: https://stackoverflow.com/a/43755216



##### 2.2 mongo 启动后自动创建用户

参考: https://stackoverflow.com/a/54064268



### 3. 参考资料

+ https://yeasy.gitbooks.io/docker_practice/compose/
+ https://juejin.im/post/5d17442e518825559f46ed92