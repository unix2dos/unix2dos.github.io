---
title: journal日志管理
tags:
  - systemctl
  - journal
categories:
  - 2-linux系统
  - 系统
abbrlink: 9a741526
date: 2021-01-29 00:00:00
---

Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用`journalctl`一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是`/etc/systemd/journald.conf`。

<!-- more -->

Systemd 的日志文件是二进制格式的，必须使用 Journald 提供的 journalctl 来查看，默认不带任何参数时会输出系统和所有后台进程的混合日志。

默认日志最大限制为所在文件系统容量的 10%，可以修改 /etc/systemd/journald.conf 中的 SystemMaxUse 来指定该最大限制。

# 1. 配置文件

默认状态所有选项均为注释状态

```bash
[Journal]
Storage=auto           #存储日志文件位置（"volatile" 表示仅保存在内存中，路径：/run/log/journal 、"persistent" 表示优先保存在磁盘上，路径：优先保存在 /var/log/journal、"auto"默认值、"none"不保存任何日志）
Compress=yes            #压缩存储大于特定阈值的对象
Seal=yes          #如果存在一个"sealing key"， 那么就为所有持久保存的日志文件启用 FSS验证
SplitMode=uid   #是否按用户进行日志分割（分割策略："uid" 表示每个用户都有专属日志文件系统用户仍写进系统日志，"none" 不进行日志分割，所有用户日志均写进系统日志
SyncIntervalSec=5m    #向磁盘刷写日志时间间隔（error以上，不含error级别日志会立即刷写）
RateLimitInterval=30s    
RateLimitBurst=1000    #上下两个选项表示在30s内每个服务最多纪录1000条日志，多余日志丢弃
SystemMaxUse=       #全部日志最大占用磁盘空间
SystemKeepFree=     #除日志文件之外，至少保留多少空间给其他用途（磁盘）
SystemMaxFileSize=     #单个日志文件最大占用磁盘空间
RuntimeMaxUse=      #全部日志最大占用内存空间
RuntimeKeepFree=    #除日志文件之外，至少保留多少空间给其他用途（内存）
RuntimeMaxFileSize=     #单个日志文件最大占用内存大小
MaxRetentionSec=      #日志文件最大保留期限（超过该时间自动删除）
MaxFileSec=1month     #日志滚动时间间隔
ForwardToSyslog=yes   #表示是否将接收到的日志消息转发给传统的 syslog 守护进程
ForwardToKMsg=no        #表示是否将接收到的日志消息转发给内核日志缓冲区(kmsg)
ForwardToConsole=no    #表示是否将接收到的日志消息转发给系统控制台
ForwardToWall=yes      #表示是否将接收到的日志消息作为警告信息发送给所有已登录用户
TTYPath=/dev/console     
MaxLevelStore=debug      #表示记录到日志中的最高日志等级
MaxLevelSyslog=debug     #转发给syslog进程的最高日志等级
MaxLevelKMsg=notice     #转发给内核日志缓冲区(kmsg)的最高日志等级
MaxLevelConsole=info     #转发给系统控制台的最高日志等级
MaxLevelWall=emerg     #置作为警告信息发送给所有已登录用户的最高日志等级
LineMax=48K     #每条日志记录最大的字节长度，超长部分自动打分割符进行分割
```



# 2. 命令

### 2.1 查看日志

```bash
# 查看所有日志（默认情况下 ，只保存本次启动的日志）
$ sudo journalctl

# 显示尾部的最新10行日志
$ sudo journalctl -n

# 显示尾部指定行数的日志
$ sudo journalctl -n 20

# 实时滚动显示最新日志
$ sudo journalctl -f

# 查看指定时间的日志
$ sudo journalctl --since="2012-10-30 18:17:16"
$ sudo journalctl --since "20 min ago"
$ sudo journalctl --since yesterday
$ sudo journalctl --since "2015-01-10" --until "2015-01-11 03:00"
$ sudo journalctl --since 09:00 --until "1 hour ago"


# 查看内核日志（不显示应用日志）
$ sudo journalctl -k

# 查看系统本次启动的日志
$ sudo journalctl -b
$ sudo journalctl -b -0

# 查看上一次启动的日志（需更改设置）
$ sudo journalctl -b -1


# 查看指定优先级（及其以上级别）的日志，共有8级 0: emerg 1: alert 2: crit 3: err 4: warning 5: notice 6: info 7: debug
$ sudo journalctl -p err -b
```



### 2.2 查看指定服务日志

```bash
# 查看指定服务的日志
$ sudo journalctl /usr/lib/systemd/systemd

# 查看指定进程的日志
$ sudo journalctl _PID=1

# 查看某个路径的脚本的日志
$ sudo journalctl /usr/bin/bash

# 查看指定用户的日志
$ sudo journalctl _UID=33 --since today

# 查看某个 Unit 的日志
$ sudo journalctl -u nginx.service
$ sudo journalctl -u nginx.service --since today

# 实时滚动显示某个 Unit 的最新日志
$ sudo journalctl -u nginx.service -f

# 合并显示多个 Unit 的日志
$ journalctl -u nginx.service -u php-fpm.service --since today
```



### 2.3 格式查看

```bash
# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager

# 以 JSON 格式（单行）输出
$ sudo journalctl -b -u nginx.service -o json

# 以 JSON 格式（多行）输出，可读性更好
$ sudo journalctl -b -u nginx.serviceqq
 -o json-pretty
```



### 2.4 查看存储

```bash
# 显示日志占据的硬盘空间
$ sudo journalctl --disk-usage

# 仅保留500MB大小的日志文
$ sudo journalctl --vacuum-size=500M

# 指定日志文件保存多久
$ sudo journalctl --vacuum-time=1years

# 仅保留最近一个月的日志文件
$ sudo journalctl --vacuum-time=1m

# 仅保留最近2天的日志文件
$ sudo journalctl --vacuum-time=2d
```



# 3. 参考资料

+ http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html