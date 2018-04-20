---
title: 'aws搭建shadowsocks梯子翻墙'
date: 2017-03-28 18:04:22
tags: linux
---

### 部署aws
1. aws免费1年,申请的时候需要信用卡,没有信用卡的可以淘宝购买虚拟信用卡

2. 申请成功,创建Ec2实例, 下载私钥, 注意保存好
3. ssh连接aws, 因为淘宝购买的信用卡只能在美国2个地区部署, 中国连接会发现特别卡顿, 可以用mosh连接,后面我会写怎么使用mosh连接
```
chmod 400 aws.pem
ssh -i "aws.pem" ubuntu@ec2-54-191-9-26.us-west-2.compute.amazonaws.com
```
4. 修改root密码
```
sudo passwd root
```
<!-- more -->
-------

### 安装shadowsocks

1. 更新apt-get
```
 apt-get update
 apt-get install python-pip //安装python-pip
```

2. 安装python-pip:
```
sudo apt-get purge python-pip
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
hash -r
```
3. 安装shadowsocks
```
pip install shadowsocks // 安装shadowsocks
ssserver -c /etc/shadowsocks.json -d start //启动shadowsocks
```
4. 配置shadowsocks
shadowsocks.json需要自己创建，默认是没有的 注意端口修改的是server_port, local_port是固定的
```
{
  "server": "0.0.0.0",
  "server_port": 6789,
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "password": "******",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false,
  "workers": 1
}
```
配置以后重启shadowsocks:
```
sudo ssserver -c /etc/shadowsocks.json -d restart
```
### 配置aws和使用shadowsocks翻墙

>安装之后，添加服务器，地址为AWS的外网地址，登录AWS控制台，查看正在运行中的实例，找到公有ip。 端口号为刚才配置Shadowsocks服务器时的端口号，密码也是刚才配置的，设置完之后保存。
![alt text][id1]

>配置好shaodowsocks后，还需要将配置中的端口打开,这样客户端的服务才能链接得上EC2中的shadowsocks服务
首先打开正在运行的实例，向右滚动表格，最后一项，安全组，点击进入，编辑入站规则，默认是开启了一个22端口（这是给ssh访问的）

![alt text][id2]
[id1]: https://github.com/unix2dos/unix2dos.github.io/blob/source/source/images/1.png
[id2]: https://github.com/unix2dos/unix2dos.github.io/blob/source/source/images/2.png
>如果不放心流量超限的话可以设置下账单报警。

