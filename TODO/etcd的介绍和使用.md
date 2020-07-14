---
title: "etcd的介绍和使用"
date: 2019-05-07 00:00:00
tags:
- golang
- etcd
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
ETCD_VER=v3.4.9

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

### 2.2 使用

为了方便, 移动到环境变量下

```bash
mv /tmp/etcd-download-test/etcd /usr/local/bin/
mv /tmp/etcd-download-test/etcdctl /usr/local/bin/
```

启动

```bash
nohup etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://0.0.0.0:2379 &
```

操作

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
```



# 3. golang 使用 etcd

```bash
go get go.etcd.io/etcd/clientv3
```









# 4. 参考资料

+ https://www.liwenzhou.com/posts/Go/go_etcd/