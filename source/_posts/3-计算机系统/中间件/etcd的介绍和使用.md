---
title: etcd的介绍和使用
tags:
  - golang
  - etcd
categories:
  - 3-计算机系统
  - 中间件
abbrlink: c0b47ece
date: 2020-09-25 00:00:00
---



# 1. etcd 介绍

[etcd](https://etcd.io/)是使用Go语言开发的一个开源的、高可用的分布式key-value存储系统，可以用于配置共享和服务的注册和发现。类似项目有zookeeper和consul。

etcd具有以下特点：

- 完全复制：集群中的每个节点都可以使用完整的存档
- 高可用性：Etcd可用于避免硬件的单点故障或网络问题
- 一致性：每次读取都会返回跨多主机的最新写入
- 简单：包括一个定义良好、面向用户的API（gRPC）
- 安全：实现了带有可选的客户端证书身份验证的自动化TLS
- 快速：每秒10000次写入的基准速度
- 可靠：使用Raft算法实现了强一致、高可用的服务存储目录

<!-- more -->

# 2. etcd 安装和使用

### 2.1 下载安装

https://github.com/etcd-io/etcd/releases

```bash
ETCD_VER=v3.4.13

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
```



### 2.2 启动

为了方便, 移动到环境变量下

```bash
mv /tmp/etcd-download-test/etcd /usr/local/bin/
mv /tmp/etcd-download-test/etcdctl /usr/local/bin/
```

+ 启动

```bash
etcd

nohup etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://0.0.0.0:2379 &
```

+ etcd TCP ports

The [official etcd ports](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt) 对于客户端请求是2379，即curl set get 时用的端口。节点间通信端口是2380。



### 2.3 操作

```bash
etcdctl put mykey "this is awesome"
#OK

etcdctl get mykey
#mykey
#this is awesome

etcdctl del mykey
#1

etcdctl get mykey
#


# 获取所有的键值
etcdctl get / --prefix --keys-only
etcdctl get "" --prefix --keys-only | sed '/^\s*$/d'
```



### 2.4 集群

```
etcd --config-file etcd.conf
```

etcd1.conf

```bash
name: etcd-1
data-dir: /data/server/etcd/data
listen-client-urls: http://172.21.0.17:2379,http://127.0.0.1:2379
advertise-client-urls: http://172.21.0.17:2379,http://127.0.0.1:2379
listen-peer-urls: http://172.21.0.17:2380 # 监听的
initial-advertise-peer-urls: http://172.21.0.17:2380
initial-cluster: etcd-1=http://172.21.0.17:2380,etcd-2=http://172.21.0.17:2390,etcd-3=http://172.21.0.15:2380
initial-cluster-token: etcd-cluster-token
initial-cluster-state: new
```

etcd2.conf

```bash
name: etcd-2
data-dir: /data/server/etcd2/data
listen-client-urls: http://172.21.0.17:2389,http://127.0.0.1:2389
advertise-client-urls: http://172.21.0.17:2389,http://127.0.0.1:2389
listen-peer-urls: http://172.21.0.17:2390 # 监听的
initial-advertise-peer-urls: http://172.21.0.17:2390
initial-cluster: etcd-1=http://172.21.0.17:2380,etcd-2=http://172.21.0.17:2390,etcd-3=http://172.21.0.15:2380
initial-cluster-token: etcd-cluster-token
initial-cluster-state: existing
```

etcd3.conf

```bash
# 节点名称
name: etcd-3 

# 数据存储目录
data-dir: /data/server/etcd/data 

# 监听在客户端流量上的URL列表，该参数告诉etcd在指定的协议://IP:port组合上接受来自客户端的传入请求。
listen-client-urls: http://172.21.0.15:2379,http://127.0.0.1:2379 

# 此成员的客户端URL的列表，这些URL广播给集群的其余部分。 这些URL可以包含域名。
advertise-client-urls: http://172.21.0.15:2379,http://127.0.0.1:2379

# 监听在对等节点流量上的URL列表，该参数告诉etcd在指定的协议://IP:port组合上接受来自其对等方的传入请求。
listen-peer-urls: http://172.21.0.15:2380

# 此成员的对等URL的列表，以通告到集群的其余部分。 这些地址用于在集群周围传送etcd数据。 所有集群成员必须至少有一个路由。 这些URL可以包含域名。
initial-advertise-peer-urls: http://172.21.0.15:2380

# 启动集群的初始化配置
initial-cluster: etcd-1=http://172.21.0.17:2380,etcd-2=http://172.21.0.17:2390,etcd-3=http://172.21.0.15:2380

# 引导期间etcd群集的初始集群令牌。
initial-cluster-token: etcd-cluster-token

# 初始群集状态（“新”或“现有”）。 对于在初始静态或DNS引导过程中存在的所有成员，将其设置为new。 如果此选项设置为existing，则etcd将尝试加入现存集群。 如果设置了错误的值，etcd将尝试启动，但会安全地失败。
initial-cluster-state: new
```



`etcdctl member list`

```
hash1: name=etcd-1 peerURLs=http://172.21.0.17:2380 clientURLs=http://127.0.0.1:2379,http://172.21.0.17:2379 isLeader=false

hash2: name=etcd-2 peerURLs=http://172.21.0.17:2390 clientURLs=http://127.0.0.1:2389,http://172.21.0.17:2389 isLeader=false

hash3: name=etcd-3 peerURLs=http://172.21.0.15:2380 clientURLs=http://127.0.0.1:2379,http://172.21.0.15:2379 isLeader=true
```



# 3. golang 使用 etcd

https://github.com/etcd-io/etcd/tree/master/client

```bash
go get go.etcd.io/etcd/v3/client
```

代码:

```go
package main

import (
	"log"
	"time"
	"context"

	"go.etcd.io/etcd/v3/client"
)

func main() {
	cfg := client.Config{
		Endpoints:               []string{"http://127.0.0.1:2379"},
		Transport:               client.DefaultTransport,
		// set timeout per request to fail fast when the target endpoint is unavailable
		HeaderTimeoutPerRequest: time.Second,
	}
	c, err := client.New(cfg)
	if err != nil {
		log.Fatal(err)
	}
	kapi := client.NewKeysAPI(c)
	// set "/foo" key with "bar" value
	log.Print("Setting '/foo' key with 'bar' value")
	resp, err := kapi.Set(context.Background(), "/foo", "bar", nil)
	if err != nil {
		log.Fatal(err)
	} else {
		// print common key info
		log.Printf("Set is done. Metadata is %q\n", resp)
	}
	// get "/foo" key's value
	log.Print("Getting '/foo' key value")
	resp, err = kapi.Get(context.Background(), "/foo", nil)
	if err != nil {
		log.Fatal(err)
	} else {
		// print common key info
		log.Printf("Get is done. Metadata is %q\n", resp)
		// print value
		log.Printf("%q key has %q value\n", resp.Node.Key, resp.Node.Value)
	}
}
```



### 3.1 报错 undefined: balancer.PickOptions

具体操作方法是在go.mod里加上：

```go
replace google.golang.org/grpc => google.golang.org/grpc v1.26.0
```



# 4. 参考资料

+ https://github.com/etcd-io/etcd
+ https://blog.pytool.com/devops/etcd/etcdctl/
+ https://www.liwenzhou.com/posts/Go/go_etcd/
+ https://www.cnblogs.com/cbkj-xd/p/11934599.html