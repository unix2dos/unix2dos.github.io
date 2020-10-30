---
title: 常用docker快速启动命令
tags:
  - docker
categories:
  - 2-linux系统
  - docker
abbrlink: 4aa4f44c
date: 2020-10-22 00:00:00
---

# 1. mysql

### 1.1 启动5.7版本

```bash
docker pull mysql:5.7
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

<!-- more -->

### 1.2 启动最新版本

```bash
docker pull mysql:latest   
docker run -p 3307:3306 --name mysql_latest -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```

看最新版本对应的版本号:

``` bash
docker run --rm mysql:latest mysql -V
```



# 2. redis

### 2.1 启动最新版本

```bash
docker pull redis:latest
docker run -d --name redis -p 6379:6379 redis:latest  --requirepass "123456"
```

看最新版本对应的版本号:

```bash
docker run --rm redis:latest redis-cli -v
```












