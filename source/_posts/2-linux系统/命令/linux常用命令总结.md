---
title: linux常用命令总结
tags:
  - linux
abbrlink: 1d063ae7
categories:
  - 2-linux系统
  - 命令
date: 2019-07-19 17:54:46
---



# 1. 查看linux系统信息

### 1.1 查看系统版本

```bash
#lsb_release -a 
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.6 LTS
Release:        16.04
Codename:       xenial

#uname -srm  
Linux 4.15.0-1060-gcp x86_64

#cat /etc/os-release
NAME="Ubuntu"
VERSION="16.04.6 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.6 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial

#cat /proc/version
Linux version 4.15.0-1060-gcp (buildd@lcy01-amd64-028) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.12)) #64-Ubuntu SMP Thu Mar 26 03:21:15 UTC 2020

#cat /etc/issue
Ubuntu 16.04.6 LTS \n \l
```

<!-- more -->

### 1.2 查看硬盘内存 CPU

##### 1.2.1 查看硬盘

```bash
# df -h  查看磁盘空间
Filesystem      Size  Used Avail Use% Mounted on
udev            986M     0  986M   0% /dev
tmpfs           200M   22M  179M  11% /run
/dev/sda1        29G  5.6G   24G  20% /


#fdisk -l  查看Linux中的所有磁盘分区
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: A03F431A-7AE6-406D-B233-EED234598EEE

Device      Start      End  Sectors  Size Type
/dev/sda1  227328 62914526 62687199 29.9G Linux filesystem
/dev/sda14   2048    10239     8192    4M BIOS boot
/dev/sda15  10240   227327   217088  106M EFI System
```

##### 1.2.1 查看内存

```bash
# free -h 查看内存
              total        used        free      shared  buff/cache   available
Mem:           1.9G        769M        211M         21M        1.0G        1.0G
Swap:            0B          0B          0B
```

##### 1.2.2 查看 CPU

```bash
# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2

# cat /proc/cpuinfo   查看 CPU 信息
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 63
model name      : Intel(R) Xeon(R) CPU @ 2.30GHz
stepping        : 0
microcode       : 0x1
cpu MHz         : 2300.000
cache size      : 46080 KB

# cat /proc/cpuinfo| grep "processor"| wc -l   查看 CPU 个数
2
```



# 2. 进程端口相关

### 2.1 查询进程

```bash
ps     # displays processes for the current shell.
ps -ef # Display every active process on a Linux system in generic (Unix/Linux) format.
ps -aux # Display all processes in BSD format.
```



### 2.2 查询端口

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



### 2.3 进程管理器

```bash
# top
top - 13:41:19 up 83 days,  2:53,  1 user,  load average: 0.21, 0.85, 0.72
Tasks: 148 total,   1 running, 106 sleeping,   0 stopped,   0 zombie
%Cpu(s):  7.0 us,  4.1 sy,  0.0 ni, 88.4 id,  0.2 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  2040116 total,   212880 free,   789056 used,  1038180 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1040512 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
30844 root      20   0  572436 296072  16972 S   4.7 14.5   4235:28 kube-apiserver
31035 root      20   0  643968  55980  20080 S   4.3  2.7   3362:32 kubelet
30735 root      20   0 10.121g  67348  10588 S   2.3  3.3   1994:44 etcd
```

+ 第一行的参数

  **10:59:22**  : 当前系统时间
   **up 37 days, 20:48**  : 系统累积以及运行的时间
   **3 users** : 当前用户数量
   **load average: 0.00,0.00,0.00** : 系统负载的三个数值分别表示的是1分钟，5分钟和15分钟系统负载的平均值

  当CPU完全空闲的时候，平均负荷为0；当CPU工作量饱和的时候，平均负荷为1。n个CPU的电脑，可接受的系统负荷最大为n.0。

+ 第二行的参数

   **Tasks： 112 total** : 进程总数
   **1 running** : 正常运行的进程数量
   **121 sleeping** : 休眠的进程数量
   **0 stopped** : 停止的进程数量
   **0 zombie** : 僵死进程数量

+ 第三行的参数

  **0.2 us** : 用户进程占用cpu资源的百分比
   **0.2 sy** : 内核进程占用cpu资源的百分比
   **0.0 ni** : 用户进程空间内改变过优先级的进程占cpu资源的百分比
   **99.7 id** : 空闲cpu百分比
   **0.0 wa** : 等待io的进程占cpu资源的百分比
   **0.0 hi** : 硬中断占用cpu的百分比
   **0.0 si**: 软中断占用的百分比
   **0.0 st** : 虚拟机占用百分比

+ 第四行的意义

  **524280k total** : 交换区内存总容量
  **0k used** : 交换区内存使用的容量
  **524280k used**: 交换区空闲的内存容量
  **848380k cached** : 缓存的交换区总量

+ 进程的意义
   **PID** : 进程id，标记唯一进程
   **USER** : 进程用户名
   **PR** : 优先级
   **NI** : nice值。负值表示高优先级，正值表示低优先级
   **VIRT** : 进程使用的虚拟内存的大小
   **RES** : 指进程除去使用交换区swap的内存，使用的物理内存的大小
   **SHR** : 进程共享内存的大小
   **S** : process status 进程状态 。 分别有D R S T Z ,分别表示不可中断的休眠、正在运行、休眠中、暂停或者跟踪状态、僵死状态
   **%CPU** : cpu的使用量占总cpu时间的百分比
   **%MEM** : 进程使用的物理内存百分比
   **TIME+** : 进程使用的CPU时间总计，精确到1/100秒
   **COMMAND** : 命令或者进程名称



# 3. 其他

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





# 5. 参考教程

+ https://linuxtools-rst.readthedocs.io/zh_CN/latest/