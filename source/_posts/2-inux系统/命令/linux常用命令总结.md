---
title: linux常用命令总结
tags:
  - linux
abbrlink: 1d063ae7
categories:
  - 2-inux系统
  - 命令
date: 2019-07-19 17:54:46
---



### 查看linux系统版本

```bash
lsb_release -a 

uname -srm  

cat /etc/os-release

cat /proc/version

cat /etc/issue
```

<!-- more -->

### 设置linux时间和时区

```bash
# 设置时间
date -s "2015-10-25 15:00:00"

#设置时区
tzselect  #命令只告诉你选择的时区的写法，并不会生效。
在.bashrc添加  export TZ='Asia/Shanghai'
```



### 域名查询

```bash
host hostname [server]
[server]：使用不是由/etc/resolv.conf文件定义的DNS服务器IP来查询某台主机的IP。

host www.baidu.com
host www.baidu.com 8.8.8.8

host google.com #Find the Domain IP Address
host -t ns google.com #Find Domain Name Servers, dns
host -t cname mail.google.com #Find Domain CNAME Record
host -t mx google.com #Find Domain MX Record
host -t txt google.com#Find Domain TXT Record
host -a google.com #Find All Information of Domain Records and Zones
```



### 观察硬盘实体使用情况

```bash
fdisk -l  #查看Linux中的所有磁盘分区

fdisk -l /dev/sda #查看特定磁盘分区

fdisk /dev/sda #输入m里面有各种操作
```



### 查询端口

```bash
#1. 这类命令一定要用sudo
#2. a: all p: procee, t:tcp, u:udp, l:正在监听的 , n: 禁止域名解析, 只显示数字ip
sudo netstat -anp | grep 80
sudo netstat -tunlp | grep 80


# 知道进程名字反查端口
ps -ef | grep processName  #得到processID
netstat -anp | grep processID #p能显示出进程名和进程id, 过滤得到端口

# 知道端口反查进程名字
sudo lsof -i :80

# 一个命令搞定
sudo netstat -anp | grep processID
sudo netstat -anp | grep processName
```



### 查询进程

```bash
ps     # displays processes for the current shell.
ps -ef # Display every active process on a Linux system in generic (Unix/Linux) format.
ps -aux # Display all processes in BSD format.
```





### 参考教程

+ [https://www.tecmint.com](https://www.tecmint.com/) 这个网站举例命令的例子
+ https://github.com/me115/linuxtools_rst