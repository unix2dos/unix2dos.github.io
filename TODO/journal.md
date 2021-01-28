



journalctl 日志管理命令
Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用journalctl一个命令，查看所有日志（内核日志和应用日志）。

配置文件

/etc/systemd/journald.conf

日志保存目录

/var/log/journal/

默认日志最大限制为所在文件系统容量的 10%，可通过/etc/systemd/journald.conf 中的 SystemMaxUse 字段来指定

该目录是 systemd 软件包的一部分。若被删除，systemd 不会自动创建它，直到下次升级软件包时重建该目录。如果该目录缺失，systemd 会将日志记录写入 /run/systemd/journal。这意味着，系统重启后日志将丢失。

journalctl -u [服务名]
查看指定单元的日志

journalctl -b
journalctl -b -0 显示本次启动的信息
journalctl -b -1 显示上次启动的信息
journalctl -b -2 显示上上次启动的信息







### **日志管理**

Systemd 通过其标准日志服务 Journald 提供的配套程序 journalctl 将其管理的所有后台进程打印到 std:out（即控制台）的输出重定向到了日志文件。

Systemd 的日志文件是二进制格式的，必须使用 Journald 提供的 journalctl 来查看，默认不带任何参数时会输出系统和所有后台进程的混合日志。

默认日志最大限制为所在文件系统容量的 10%，可以修改 /etc/systemd/journald.conf 中的 SystemMaxUse 来指定该最大限制。

```javascript
# 查看所有日志（默认情况下 ，只保存本次启动的日志）
$ sudo journalctl

# 查看内核日志（不显示应用日志）：--dmesg 或 -k
$ sudo journalctl -k

# 查看系统本次启动的日志（其中包括了内核日志和各类系统服务的控制台输出）：--system 或 -b
$ sudo journalctl -b
$ sudo journalctl -b -0

# 查看上一次启动的日志（需更改设置）
$ sudo journalctl -b -1

# 查看指定服务的日志：--unit 或 -u
$ sudo journalctl -u docker.servcie

# 查看指定服务的日志
$ sudo journalctl /usr/lib/systemd/systemd

# 实时滚动显示最新日志
$ sudo journalctl -f

# 查看指定时间的日志
$ sudo journalctl --since="2012-10-30 18:17:16"
$ sudo journalctl --since "20 min ago"
$ sudo journalctl --since yesterday
$ sudo journalctl --since "2015-01-10" --until "2015-01-11 03:00"
$ sudo journalctl --since 09:00 --until "1 hour ago"

# 显示尾部的最新 10 行日志：--lines 或 -n
$ sudo journalctl -n

# 显示尾部指定行数的日志
$ sudo journalctl -n 20

# 将最新的日志显示在前面
$ sudo journalctl -r -u docker.service

# 改变输出的格式：--output 或 -o
$ sudo journalctl -r -u docker.service -o json-pretty

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

# 查看指定优先级（及其以上级别）的日志，共有 8 级
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug
$ sudo journalctl -p err -b

# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager

# 以 JSON 格式（单行）输出
$ sudo journalctl -b -u nginx.service -o json

# 以 JSON 格式（多行）输出，可读性更好
$ sudo journalctl -b -u nginx.serviceqq
 -o json-pretty

# 显示日志占据的硬盘空间
$ sudo journalctl --disk-usage

# 指定日志文件占据的最大空间
$ sudo journalctl --vacuum-size=1G

# 指定日志文件保存多久
$ sudo journalctl --vacuum-time=1years
```