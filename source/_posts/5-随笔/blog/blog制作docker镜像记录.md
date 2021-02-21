---
title: blog制作docker镜像记录
tags:
  - blog
categories:
  - 5-随笔
  - blog
abbrlink: 252a8c24
date: 2021-02-21 00:00:00
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



# 2. 镜像准备

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

# 安装 node 模块
npm i

# 分享按钮
git clone https://github.com/theme-next/theme-next-needmoreshare2 source/lib/needsharebutton  
# 丝带
git clone https://github.com/theme-next/theme-next-canvas-ribbon source/lib/canvas-ribbon
# 蜘蛛网
git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest
# 三种特效
git clone https://github.com/theme-next/theme-next-three source/lib/three 
# 特殊汉字
git clone https://github.com/theme-next/theme-next-han source/lib/Han
# 快速点击
git clone https://github.com/theme-next/theme-next-fastclick source/lib/fastclick
# 懒加载
git clone https://github.com/theme-next/theme-next-jquery-lazyload source/lib/jquery_lazyload
# 顶部的进度
git clone https://github.com/theme-next/theme-next-pace source/lib/pace 
# 图片展示
git clone https://github.com/theme-next/theme-next-fancybox3 source/lib/fancybox 
# 文字显示加空格
git clone https://github.com/theme-next/theme-next-pangu.git source/lib/pangu
# 读取进度
git clone https://github.com/theme-next/theme-next-reading-progress source/lib/reading_progress



# check
hexo server --config source/_data/next.yml

# deploy
hexo clean --config source/_data/next.yml && hexo g -d --config source/_data/next.yml
```



# 3. 构造镜像

Docker Hub 免费版只有1个私有库，5个私有库要7美元一个月（倒也不是给不起这个钱，只是确实没钱....）

[自己搭 Docker Registry](https://docs.docker.com/registry/) 又嫌麻烦, 就放到国内的免费云服务器商了

### 3.1 从容器构建镜像

```bash
docker commit -a "levonfly" -m "my blog images" a404c6c174a2  levonfly/blog
```

### 3.2 上传

https://cr.console.aliyun.com/

```bash
docker login --username=l6241425 registry.cn-beijing.aliyuncs.com
docker tag f71940a6db66 registry.cn-beijing.aliyuncs.com/levonfly/blog:1.0
docker push registry.cn-beijing.aliyuncs.com/levonfly/blog:1.0
```

### 3.3 使用

```bash
docker run -itd --name blog registry.cn-beijing.aliyuncs.com/levonfly/blog:1.0
docker exec -it blog /bin/bash  
```



# 4.参考资料

+ https://1c7.me/2019-1-31-china-free-private-docker-registry/