---
title: "blog制作docker镜像记录"
date: 2021-02-21 00:00:00
tags:
- blog
---

# 1. 宿主机准备

因为 blog 涉及的本地依赖过多, 特意放到 docker 上, 方便移植. 为了方便制作镜像（下载速度），在海外服务器上进行制作。

<!-- more -->

### 1.1 google云(免费撸的)

```bash
sudo passwd root

sudo vi /etc/ssh/sshd_config  #两个 yes
PermitRootLogin
PasswordAuthentication

sudo systemctl restart sshd 
```

通过 root@外网 ip 进行连接即可



### 1.2 安装docker

```bash
apt update
apt install docker.io
```



# 2. docker镜像制作

### 2.1 创建ubuntu 容器

```bash
docker run -itd --name blog ubuntu
docker exec -it blog /bin/bash  

apt update
apt install -y vim
apt install -y git
```



### 2.2 hexo 环境准备

```bash
apt install nodejs
apt install npm
npm install hexo-cli -g


#  git 中文显示
git config --global core.quotepath false        
git config --global gui.encoding utf-8   
git config --global i18n.commit.encoding utf-8  
git config --global i18n.logoutputencoding utf-8
export LESSCHARSET=utf-8
```



### 2.4 blog 环境准备

```bash
git clone git@github.com:unix2dos/unix2dos.github.io.git
git submodule update --init
npm i

# check
hexo server --config source/_data/next.yml

# deploy
hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
```



### 2.5 构造镜像

```bash

```

