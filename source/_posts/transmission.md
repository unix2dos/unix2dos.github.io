---
title: "transmission编译安装和golang_rpc的调用"
date: 2018-07-08 16:38:13
tags:
- golang
- torrent
- c
- transmission
---

### 1. mac编译transmission

+ 下载项目

```bash
git clone https://github.com/transmission/transmission Transmission
cd Transmission
git submodule update --init
Xcode project file (Transmission.xcodeproj) for building in Xcode. 
```



+ 在 xcode中编译

  下图第一个是编译 mac 的应用程序,  第二个是可以编译 transmission-daemon 程序

![1](transmission/1.png)

<!-- more -->



### 2. ubuntu 16.04编译transmission

```bash
$ sudo apt-get install cmake make build-essential automake autoconf libtool pkg-config intltool libcurl4-openssl-dev libglib2.0-dev libevent-dev libminiupnpc-dev libgtk-3-dev libappindicator3-dev gettext libssl-dev

$ git clone https://github.com/transmission/transmission Transmission
$ cd Transmission
$ git submodule update --init
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install (make install DESTDIR=a)
```

安装完成后出现以下命令:

![2](transmission/2.png)



编译时遇到的问题:

1. CMAKE_MAKE_PROGRAM is not set

```
sudo apt-get install gettext
sudo apt-get install make 
sudo apt-get install libssl-dev
```

2. missing: CURL_LIBRARY CURL_INCLUDE_DIR

```
sudo apt-get install libcurl4-openssl-dev//ubuntu
yum install curl-devel//centos
```

3. autogen.sh: not found

```
sudo apt-get install autoconf
sudo apt-get install automake
sudo apt-get install libtool
```

4. transmission libevent 变成了event

   把所有的依赖全部装一遍, 安装后删除build. 重新cmake一下



### 3. transmission 介绍

- transmission-cli： 独立的命令行客户端。
- transmission-create： 用来建立.torrent种子文件的命令行工具。
- transmission-daemon： 后台守护程序。
- transmission-edit： 用来修改.torrent种子文件的announce URL。
- transmission-remote： 控制daemon的程序。
- transmission-show：查看.torrent文件的信息。



1. 配置文件目录里面包含如下一些文件：

- settings.json： 主要的配置文件，设置daemon的各项参数，包括RPC的用户名密码配置。它实际上是一个符号链接，指向的原始文件是/etc/transmission-daemon/settings.json。里面的参数解释可以参考官网的配置说明。
- torrents/： 用户存放.torrent种子文件的目录,凡是添加到下载任务的种子，都存放在这里。.torrent的命名包含,种子文件本身的名字和种子的SHA1 HASH值。
- resume/： 该存放了.resume文件，.resume文件包含了一个种子的信息，例如该文件哪些部分被下载了，下载的数据存储的位置等等。
- blocklists/： 存储被屏蔽的peer的地址。
- dht.dat： 存储DHT节点信息。



配置主要是通过修改 `/var/lib/transmission-daemon/info/settings.json` 文件中的参数来实现的。 **注意**：在编辑 transmission 配置文件的时候，需要先关闭 daemon 进程，否则编辑的参数将会被恢复到原来的状态。



2. RPC参数介绍: 

```
{
"download-dir": "/down", #下载目录的绝对路径
"incomplete-dir": "/down/temp", #临时文件路径
"rpc-authentication-required": true, #启用验证
"rpc-bind-address": "0.0.0.0", #允许任何IP通过RPC协议访问
"rpc-enabled": true, #允许通过RPC访问
"rpc-password": "123456", #RPC验证密码（保存并启动后daemon会计算并替换为HASH值以增加安全性）
"rpc-port": 9091, #RPC端口
"rpc-username": "transmission", #RPC验证用户名
"rpc-whitelist": "*", #RPC访问白名单
"rpc-whitelist-enabled": false, #关闭白名单功能以便公网访问
}
```

更多参数说明请见[官方Wiki](https://github.com/transmission/transmission/wiki/Editing-Configuration-Files)



### 4. mac使用web界面控制transmission daemon



+ 运行Xcode 编译好的客户端, 设置 Remote



![2](transmission/3.png)





在浏览器中访问`http://localhost:9091/transmission/web` 并输入设置的用户名及密码就可以看到如下界面

![2](transmission/4.png)

+ 运行Xcode编译好的transmission-daemon

 

配置文件在 `/Users/liuwei/Library/Application\ Support/transmission-daemon/settings.json` 

设置环境变量后 `export TRANSMISSION_WEB_HOME=/Users/liuwei/workspace/transmission/web`

通过浏览器访问`http://localhost:9091/transmission/web`



+ 访问外网ip错误

unauthorized ip address403: ForbiddenUnauthorized IP Address.Either disable the IP address whitelist or add your address to it.If you're editing settings.json, see the 'rpc-whitelist' and 'rpc-whitelist-enabled' entries.If you're still using ACLs, use a whitelist instead. See the transmission-daemon manpage for details.



```
transmission/.config/transmission-daemon/settings.json  

"rpc-whitelist-enabled": true,  ture改成false。
```



### 5. golang通过rpc调用transmission



+ rpc api

https://github.com/transmission/transmission/blob/master/extras/rpc-spec.txt

+ golang lib for Transmission API

https://github.com/pyed/transmission