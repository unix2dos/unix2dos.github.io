---
title: python爬虫在docker中的实践
tags:
  - docker
  - python
  - 爬虫
categories:
  - 2-linux系统
  - docker
abbrlink: dc81a411
date: 2020-04-25 12:00:00
---



### 1. 选择镜像

这里选择基础镜像时是有讲究. 一是应当尽量选择官方镜像库里的基础镜像；二是应当选择轻量级的镜像做底包.

就典型的 Linux 基础镜像来说，大小关系如下：Ubuntu > CentOS > Debian> Alpine

Alpine Docker 镜像也继承了 Alpine Linux 发行版的这些优势。相比于其他 Docker 镜像，它的容量非常小，仅仅只有 5 MB 左右（对比 Ubuntu 系列镜像接近 200 MB），且拥有非常友好的包管理机制apk。

<!-- more -->

### 2. 拷贝文件

相对于 ADD,优先使用 COPY指令

另外发现拷贝文件夹是把文件夹的内容拷贝进去, 而不是把整个目录拷贝进去, 坑爹 最后使用dockerignore解决这个问题.

```dockerfile
copy . /zk8/
```

.dockerignore文件

```
chromedriver
*.sh
.*
**/__pycache__/
Dockerfile
```



### 3. 测试 dockerfile

##### 3.1 镜像加速

在本地测试的时候, 发现连Alpine都拉取不下来, 此处感谢伟大的 great wall. 于是选择阿里云加速.

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中

将 https://xxxxxxxxx.mirror.aliyuncs.com 加到 "registry-mirrors" 的数组里，点击Apply & Restart 按钮，等待 Docker 重启并应用配置的镜像加速器。

ps: 就是阿里云加速, 在后续的安装软件中也是特别慢, 此处建议在云服务器上(免费的谷歌云)操作.



##### 3.2 测试

```bash
docker build -t zk8:0.1 .   #  制作 image
docker run -ti --rm zk8:0.1 /bin/sh   # 启动容器结束后删除, 用这种方法可以非常方便测试
```

前期可以通过 shell 进入到容器里面测试, 在里面尝试安装相应的软件包, 然后再写 dockerfile会比较方便



### 4. 安装软件包

爬虫用 python 写的, 并且使用了 selenium + 无头浏览器. 所以安装包要写在 dockerfile里, 文件如下:

```dockerfile
FROM alpine

RUN mkdir -p /zk8
COPY . /zk8/

# install python
RUN echo "**** install python ****" && \
                apk add --no-cache python3 && \
                if [ ! -e /usr/bin/python ]; then ln -sf python3 /usr/bin/python ; fi && \
                echo "**** install pip ****" && \
                python3 -m ensurepip && \
                rm -r /usr/lib/python*/ensurepip && \
                pip3 install --no-cache --upgrade pip setuptools wheel && \
                if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi

# install python package
RUN apk add --no-cache py-lxml && \
                apk add --no-cache chromium && \
                apk add --no-cache chromium-chromedriver && \
                if [ -e /usr/bin/chromedriver ]; then ln -s /usr/bin/chromedriver /zk8/chromedriver ; fi && \
                pip install selenium && \
                pip install bearychat && \
                pip install pyquery


WORKDIR /zk8
CMD python3 main.py
```



### 5. 发布到 dockerhub

建议建立自己的私有仓库, 因为 dockerhub 可以免费使用一个私有仓库, 此处上传到 dockerhub.

```bash
docker login # 登录自己的 dockerhub 帐号

docker tag zk8:0.1 levonfly/zk8:0.1 # 此处打 tag, 格式要以 用户名/镜像名字:版本号

docker push levonfly/zk8:0.1 # 推送到 dockerhub
```

dockerhub 上还可以 link 到github, 即 github 一更新代码就重新 build.

接下来就是激动人心的时刻, 在任何安装 docker 的机器上直接运行自己的爬虫.

```bash
docker pull levonfly/zk8:0.1
docker run -d --name zk8 levonfly/zk8:0.1 
```





