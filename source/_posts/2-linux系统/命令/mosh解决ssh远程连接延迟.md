---
title: mosh解决ssh远程连接延迟
tags: linux
abbrlink: 21b6636c
categories:
  - 2-linux系统
  - 命令
date: 2017-04-06 17:54:46
---


### 安装mosh
```
sudo apt-get install mosh //server
brew install mobile-shell //mac
```


### 需要先设置本地
locale-gen zh_CN.UTF-8

### 远程服务器开启 mosh-server

### 需要aws开启udp mosh的端口

### 客户端连接

```
 mosh ubuntu@ec2-54-191-9-26.us-west-2.compute.amazonaws.com -ssh="ssh -i 'aws.pem'"
```
