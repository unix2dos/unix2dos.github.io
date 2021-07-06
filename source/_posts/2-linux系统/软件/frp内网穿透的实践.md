---
title: frp内网穿透的实践
tags:
  - frp
  - linux
categories:
  - 2-linux系统
  - 系统
abbrlink: cc6fa0e8
date: 2019-12-13 21:11:46
---



### 0. 为什么内网穿透

从公网中访问自己的私有设备向来是一件难事儿。

自己的主力台式机、NAS等等设备，它们可能处于路由器后，或者运营商因为IP地址短缺不给你分配公网IP地址。如果我们想直接访问到这些设备（远程桌面，远程文件，SSH等等），一般来说要通过一些转发或者P2P组网软件的帮助。

<!-- more -->



### 1. 安装配置 frp

https://github.com/fatedier/frp/releases 下载最新的release

##### 1.1服务端配置

```bash
sudo cp systemd/frps.service /lib/systemd/system/

#  /usr/bin/frps -c /etc/frp/frps.ini
# 我们按照 service 内的配置把文件拷贝到相应的地方

sudo cp frps /usr/bin/frps
sudo mkdir /etc/frp/
sudo cp frps.ini /etc/frp/frps.ini

# 编辑frps.ini


# 开启 service
sudo systemctl start frps
sudo systemctl enable frps
```



##### 1.2 客户端

```bash
sudo cp systemd/frpc.service /lib/systemd/system/

sudo cp frpc /usr/bin/frpc
sudo mkdir /etc/frp/
sudo cp frpc.ini /etc/frp/frpc.ini

#编辑frpc.ini

#1. 修改server_addr为服务端公网IP


# 开启 service
sudo systemctl start frpc
sudo systemctl enable frpc
```



##### 1.3 测试

主题frpc里面有ssh配置, 所以我们通过 ssh 访问内网机器：

```bash
ssh -p 6000 root@49.234.15.70

# 为了方便, 我们可以在~/.ssh/config 配置下面的话

host box
    Hostname 49.234.15.70
    Port 6000
    user root
```



### 2. 自定义域名访问内网的 web 服务

有时想要让其他人通过域名访问或者测试我们在本地搭建的 web 服务，但是由于本地机器没有公网 IP，无法将域名解析到本地的机器，通过 frp 就可以实现这一功能，以下示例为 http 服务，https 服务配置方法相同， vhost_http_port 替换为 vhost_https_port， type 设置为 https 即可。



##### 2.1 修改 frps.ini

设置 http 访问端口为 8080：

```ini
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 8080
```



##### 2.2 启动 frps：

```bash
sudo systemctl start frps
```



##### 2.3 修改frpc.ini

修改 frpc.ini 文件，假设 frps 所在的服务器的 IP 为 x.x.x.x，local_port 为本地机器上 web 服务对应的端口, 绑定自定义域名 `www.yourdomain.com`:



```ini
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.yourdomain.com
```



##### 2.4 启动 frpc：

```bash
sudo systemctl start frpc
```



##### 2.5 绑定域名映射

将 `www.yourdomain.com` 的域名 A 记录解析到 IP `x.x.x.x`，如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名。

通过浏览器访问 `http://www.yourdomain.com:8080` 即可访问到处于内网机器上的 web 服务。



### 3. frp 管理面板

服务端 frps 配置如下:

```ini
[common]
bind_port = 7000
dashboard_port = 5000
dashboard_user = admin
dashboard_pwd = admin
vhost_http_port = 5001
```

然后我们通过 ip:5000,即可以访问到 web 管理界面.



### 4. nginx 反向代理进行无端口访问

##### 4.1 frp 相应配置

+ frps

  ```ini
  [common]
  bind_port = 7000
  dashboard_port = 5000
  dashboard_user = admin
  dashboard_pwd = admin
  vhost_http_port = 5001
  ```

  

+ frpc

  ```ini
  [web]
  type = http
  local_port = 80
  custom_domains = box.frp.liuvv.com
  ```



##### 4.2 在服务端架设 nginx

1、 frp.liuvv.com 做A记录，解析至IP；

2、 *.frp.liuvv.com 做CNAME记录，解析至 frp.liuvv.com;

3、 配置nginx反向代理,将来自*.frp.liuvv.com的80端口请求，分发至frp服务器http请求的监听端口。

```nginx
server {

    listen 80;

    server_name frp.liuvv.com;

    location / {

        proxy_pass http://127.0.0.1:5000;# dashboard

        proxy_set_header    Host            $host:80;

        proxy_set_header    X-Real-IP       $remote_addr;

        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_hide_header   X-Powered-By;

    }

}

server {

    listen 80;

    server_name *.frp.liuvv.com;

    access_log /var/log/nginx/frp_access.log;

    error_log /var/log/nginx/frp_error.log;

    location / {

        proxy_pass http://127.0.0.1:5001;# vhost_http

        proxy_set_header    Host            $host:80;

        proxy_set_header    X-Real-IP       $remote_addr;

        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_hide_header   X-Powered-By;

    }

}
```



### 5.  远程桌面连接局域网mac

mac去下载darwin_amd64,  然后启动客户端 frpc

##### 5.1  开启屏幕共享

在系统设置->共享->开启屏幕共享

即开启了 vnc 服务



##### 5.2 修改frpc配置

```ini
[common] 
server_addr = 49.234.15.70
server_port = 7000

[vnc] 
type = tcp 
local_ip = 127.0.0.1 
local_port = 5900 
remote_port = 35900 
use_encryption = true 
use_compression = true
```



##### 5.3 连接远程

在 finder, cmd+k, 进行连接

```ini
vnc://49.234.15.70:35900
```



### 6. 参考资料:

+ https://github.com/fatedier/frp/blob/master/README_zh.md

+ https://www.iyuu.cn/archives/286/
+ [实现MAC远程桌面](http://yuqiangcoder.com/2019/11/22/frp-内网穿透-实现MAC远程桌面.html)