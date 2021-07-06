---
title: supervisor进程管理使用
tags:
  - supervisor
categories:
  - 2-linux系统
  - 命令
abbrlink: 1825ecdb
date: 2020-08-23 00:00:01
---

supervisor是一个C/S系统,它可以在类UNIX系统上控制系统进程，由python编写，提供了大量的功能来实现对进程的管理。

<!-- more -->

# 1. supervisor 介绍

supervisor主要包含以下四个部分：

+ supervisord：

  这个是supervisor服务的主要管理器，负责管理我们配置的子进程，包括重启崩溃或异常退出的子进程，同时也响应来自客户端的请求。

+ supervisorctl：

  supervisord服务的客户端命令行。听过这个，我们可以获得由主进程控制的子进程的状态，停止和启动子进程，并获得主进程的运行列表。

+ Web Server：

  supervisorctl功能娉美。这个是通过web界面查看和控制进程状态。

+ XML-RPC Interface：

  服务于web UI的同一个HTTP服务器提供一个XML-RPC接口，可以用来询问和控制管理程序及其运行的程序



# 2. supervisor安装

```bash
# 安装
sudo pip install supervisor

# 启动服务
supervisord -c /etc/supervisord.conf

# 查看是否启动
ps -ef | grep supervisord
```



# 3. supervisor配置

supervisor 是一个 C/S 模型的程序，supervisord是 server 端，对应的 client 端：supervisorctl

### 3.1 默认配置

运行`echo_supervisord_conf` 命令输出默认的配置项, 默认 supervisor 会使用 /etc/supervisord.conf 作为默认配置文件。

文件内可以看到如下配置, 所以自定义的配置文件可以放到`/etc/supervisor/conf.d/`文件夹下.

```ini
[include]
files = /etc/supervisor/conf.d/*.conf
```



### 3.2 应用程序配置

```ini
[program:usercenter]  # usercenter 是应用程序的唯一标识，不能重复。对该程序的所有操作（start, restart 等）都通过名字来实现。
directory = /home/leon/projects/usercenter ; 程序的启动目录
command = gunicorn -w 8 -b 0.0.0.0:17510 wsgi:app  ; 启动命令
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数, 需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录
stdout_logfile = /data/logs/usercenter_stdout.log
```



### 3.3 使用

+ 更新配置, 修改配置后需要更新

  ```bash
supervisorctl update
  ```
  
+ 查看log, 直接查看配置里的log 即可

  ```bash
stdout_logfile=/home/supervisor_log/uwsgi.log            
  stderr_logfile=/home/supervisor_log/uwsgi.log
  ```



# 4. supervisor 常用命令

```bash
supervisorctl update #更新新的配置到supervisord（不会重启原来已运行的程序）
supervisorctl reload #载入所有配置文件，并按新的配置启动、管理所有进程（会重启原来已运行的程序）

supervisorctl start xxx #启动某个进程
supervisorctl restart xxx #重启某个进程
supervisorctl stop xxx #停止某一个进程(xxx)，xxx为[program:theprogramname]里配置的值
supervisorctl stop groupworker #重启所有属于名为groupworker这个分组的进程(start,restart同理)
supervisorctl stop all #停止全部进程，注：start、restart、stop都不会载入最新的配置文

supervisorctl reread #当一个服务由自动启动修改为手动启动时执行一下就ok
```



# 5. systemd介绍

> Systemd是Linux中的No.1进程, 可以用 systemd 管理 supervisor

Linux 操作系统的启动首先从 BIOS 开始，然后由 Boot Loader 载入内核，并初始化内核。内核初始化的最后一步就是启动 init 进程。这个进程是系统的第一个进程，PID 为 1，又叫超级进程，也叫根进程。它负责产生其他所有用户进程。所有的进程都会被挂在这个进程下，如果这个进程退出了，那么所有的进程都被 kill 。如果一个子进程的父进程退了，那么这个子进程会被挂到 PID 1 下面。

2010的有一天，一个在 RedHat工作的工程师 Lennart Poettering 和 Kay Sievers ，开始引入了一个新的 init 系统—— systemd。这是一个非常非常有野心的项目，这个项目几乎改变了所有的东西，systemd 不但想取代已有的 init 系统，而且还想干更多的东西。

2014 年，Debian Linux 因为想准备使用 systemd 来作为标准的 init 守护进程来替换 sysvinit 。而围绕这个事的争论达到了空前的热度，争论中充满着仇恨，systemd 的支持者和反对者都在互相辱骂，导致当时 Debian 阵营开始分裂。

今天，systemd 占据了几乎所有的主流的 Linux 发行版的默认配置，包括：Arch Linux、CentOS、CoreOS、Debian、Fedora、Megeia、OpenSUSE、RHEL、SUSE企业版和 Ubuntu。而且，对于 CentOS, CoreOS, Fedora, RHEL, SUSE这些发行版来说，不能没有 systemd。



# 6. supervisor和 systemd 对比

### 6.1 supervisor

+ 优点

  1 可以通过网页执行启动停止的操作
  2 单配置文件可控制多个程序
  3 可控制进程数量
  4 进程资源控制能力比较强

+ 缺点

  1 本身需要被监控
  2 开机自启依赖其他程序
  3 不能跨主机
  4 依赖于meld3、setuptools
  5 进程需在前台运行



### 6.2 systemd

+ 优点

  1可使用模板文件
  2 附带定时器、路径监控器、数据监控器等功能
  3 比较弱的跨主机能力，节点必须互相添加ssh key信任，只能远程控制已有的服务
  4 开机可以自启
  5 大多数发行版的标准配置
  6 配套journalctl二进制保存日志很难伪造，日志格式统一，日志大小可限制
  7 限制特定服务可用的系统资源量例如CPU、程序堆栈、文件句柄数量、子进程数量

+ 缺点

  1 多配置文件才能配置多个程序

  

# 7. 参考资料

+ http://supervisord.org/index.html
+ https://woni.link/post/12
+ https://coolshell.cn/articles/17998.html