---
title: QQ机器人酷Q的使用
tags:
  - qq
  - docker
  - robot
abbrlink: 545a4e78
date: 2019-03-31 12:00:00
---



近期准备用qq机器人实现往qq群里发消息. 需要用到qq机器人.

据说在2019年前, 用qq机器人是非常之方便. 但是自从Smart QQ 协议在 2019 年 1 月 1 日停止服务后, 网上好多qq机器人项目都失效了.

目前找到了一款酷Q机器人 https://cqp.cc/, 使用并且测试成功.  最重要的一点是酷Q的Air版还是免费的.



<!-- more -->

### 1. 下载使用

酷Q官方下载的是windows版本(https://cqp.cc/t/23253) , 需要在windows上运行并登录QQ. 这个虽然简单方便, 但是需要一直在windows上挂着, 很显然这个条件不太具备.

目的是在linux上挂机运行酷Q, 所以找到了酷Q的`docker`版本.



### 2. 安装酷Q docker版本

+ 安装docker(已安装docker直接看下一步)

```shell
yum install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce


systemctl start docker
systemctl enable docker
```



+ 安装酷Q官方 docker 版本  https://cqp.cc/t/34558 (建议安装下面的CoolQ插件版本)

```shell
docker pull coolq/wine-coolq
mkdir /root/coolq-data # 用于存储酷 Q 的程序文件

docker run --name=coolq --d \
-p 9001:9000 \ # noVNC 端口，用于从浏览器控制酷 Q
-v /root/coolq-data:/home/user/coolq \ # 将宿主目录挂载到容器内用于持久化酷 Q 的程序文件
-e VNC_PASSWD=12345678 \ # 设置noVNC登录密码, 默认密码是 MAX8char
-e COOLQ_ACCOUNT=123456 \ # 要登录的 QQ 账号，可选
coolq/wine-coolq
```

此时在浏览器中访问 http://你的服务器IP:你的端口 即可看到远程操作登录页面，输入密码，即可看到 酷Q Air 的登录界面啦。



+ 安装CoolQ插件的 docker 版本 https://github.com/richardchien/coolq-http-api

CoolQ插件通过 HTTP 或 WebSocket 对酷 Q 的事件进行上报以及接收请求来调用酷 Q 的 DLL 接口，从而可以使用其它语言编写酷 Q 插件。

  

```shell
docker pull richardchien/cqhttp:latest
mkdir coolq  # 用于存储酷 Q 的程序文件

docker run -ti -d --name cqhttp-test \
-v $(pwd)/coolq:/home/user/coolq \  # 将宿主目录挂载到容器内用于持久化酷 Q 的程序文件
-p 9001:9000 \ # noVNC 端口，用于从浏览器控制酷 Q
-p 5700:5700 \ # HTTP API 插件开放的端口
-e VNC_PASSWD=12345678 \ # 设置noVNC登录密码, 默认密码是 MAX8char
-e COOLQ_ACCOUNT=123456 \ # 要登录的 QQ 账号，可选
-e CQHTTP_SERVE_DATA_FILES=yes \ # 允许通过 HTTP 接口访问酷 Q 数据文件
richardchien/cqhttp:latest
```



### 3. 发送消息到qq群

其实QQ机器人不仅能发送消息到qq群, 还能发送消息到个人, 转发群消息, 加好友, 踢人等一系列操作

详见api列表:  https://richardchien.gitee.io/coolq-http-api/docs/4.8/#/API



+ 调用方式也很简单, 参见api文档

```
POST  http://ip:5700/send_group_msg

group_id: qq群号
message: 发送的消息
```



### 4. 酷Q机器人AI聊天

机器人还有个用处就是可以实现AI自动聊天.

目前酷Q支持 图灵机器人(http://www.turingapi.com/)和小i机器人(http://cloud.xiaoi.com/)

安装酷Q后,添加对应的程序即可