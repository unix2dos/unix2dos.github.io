---
title: "linux源码安装python3"
date: 2019-03-31 12:02:01
tags:
- python
- linux
---



linux下大部分系统默认自带python2.x的版本. 默认的python被系统很多程序所依赖，比如centos下的yum就是python2写的，所以默认版本不要轻易删除，否则会有一些问题.

如果需要使用最新的Python3那么我们可以编译安装源码包到独立目录，这和系统默认环境之间是没有任何影响的，python3和python2两个环境并存即可

作为作死小能手, 不装最新版本怎么能行? 所以手动编译python3源码进行安装, 并记录遇到的一些问题.

<!-- more -->



### 1. 下载源码:

https://www.python.org/downloads/source/

找到最新的Source release 下载即可



### 2. 安装准备工作:

事实证明下面的这些软件不安装, 在编译python3时会出现各种问题, 所以先把这些软件都安装了.

```shell
yum install gcc  

yum install zlib* # 注意此处带*, 因为需要 zlib-devel

yum install libffi-devel # 不安装报错ModuleNotFoundError: No module named '_ctypes'

yum install openss openssl-devel # 不安装会导致pip3缺少ssl, 无法下载包

```



### 3. 源码安装python3.7

```shell
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz  # 下载源码

tar -zxvf Python-3.7.3.tgz

cd Python-3.7.3

./configure --with-ssl --with-ensurepip=install # --with-ensurepip=install是安装pip3 --with-ssl是让pip3支持ssl

make 

make altinstall # 注意此处是altinstall, 如果是install, 会和python2造成冲突
```



### 4. 建立python软链接

安装成功后, 执行程序为python3.7, 我们为了方便使用需要建立一个软链接

```shell
which python3.7
/usr/local/bin/python3.7


ln -s /usr/local/bin/python3.7 /usr/local/bin/python3
ln -s /usr/local/bin/pip3.7 /usr/local/bin/pip3
```

