---
title: goproxy的部署实践
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: 74b41bb2
date: 2019-11-29 21:11:46
---

### 0. 前言

在大陆地区我们无法直接通过 `go get` 命令获取到一些第三方包，最常见的就是 `golang.org/x` 下面的各种优秀的包. 解决方案如下:

```bash
# go.1.12.x
export GO111MODULE=on
export GOPROXY=https://goproxy.cn

# go1.13.x
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOPRIVATE=*.corp.example.com 

#GOPRIVATE=*.corp.example.com 表示所有模块路径以 corp.example.com 的下一级域名 (如 team1.corp.example.com) 为前缀的模块版本都将不经过 Go module proxy 和 Go checksum database，需要注意的是不包括 corp.example.com 本身。
```

<!-- more -->



本文将重点介绍 go module 的 proxy 配置实现，包括如下两种的代理配置：

- GOPROXY
- Athens

### 1.  goproxy

https://github.com/goproxyio/goproxy



##### 1.1 安装go

```bash
wget https://dl.google.com/go/go1.13.4.linux-amd64.tar.gz #下载go
sudo tar -C /usr/local -xzf go1.13.4.linux-amd64.tar.gz # 解压到/usr/local
export PATH=$PATH:/usr/local/go/bin # 设置环境变量
go version # go version go1.13.4 linux/amd64
```



##### 1.2 安装goproxy

```bash
git clone https://github.com/goproxyio/goproxy.git
cd goproxy/
make
```



##### 1.3 代理

```bash
# 服务端执行
./bin/goproxy -cacheDir=/tmp/test -listen=0.0.0.0:8082 


# 客户端操作
GOPROXY=http://服务端ip:8082 go get -v github.com/spf13/cobra # 会缓存在服务端/tmp/test目录下
```



##### 1.4 nginx配置

```nginx
./bin/goproxy -cacheDir=/tmp/test -listen :8082 -proxy https://goproxy.cn -exclude liuvv.com


server {
    server_name goproxy.liuvv.com;
    listen 80 ;
    listen 443 ssl http2 ;
    access_log /var/log/nginx/goproxy_access_log;
    error_log /var/log/nginx/goproxy_error_log notice;

    location  / {
        proxy_set_header       X-Real-IP $remote_addr;
        proxy_set_header       X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header       Host $http_host;
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;

        proxy_pass             http://127.0.0.1:8082;
    }
}
```



### 2.  Athens

https://github.com/gomods/athens



```bash
git clone https://github.com/gomods/athens 
cd athens 

make build-ver VERSION="0.7.0" # 如果下载不下来, 修改makefile的goproxy
./athens -version
```



服务器启动

```bash
export ATHENS_STORAGE_TYPE=disk  # 缓存到硬盘
export ATHENS_DISK_STORAGE_ROOT=~/athens-storage
./athens
```



客户端测试

```bash
export GO111MODULE=on export GOPROXY=http:#服务器ip:3000


git clone https://github.com/athens-artifacts/walkthrough.git 
cd walkthrough
go run .  # 会缓存在服务端~/athens-storage目录下


curl 服务器ip:3000/github.com/athens-artifacts/samplelib/@v/list
```



### 3. 参考资料

+ [Hello，Go module proxy](https://tonybai.com/2018/11/26/hello-go-module-proxy/)
+ [Go Module Proxy](https://juejin.im/post/5c8f9f8ef265da612c3a34b9)
+ https://blog.wolfogre.com/posts/golang-package-history/
+ https://juejin.im/post/5d8ee2db6fb9a04e0b0d9c8b
+ https://github.com/goproxy/goproxy.cn/