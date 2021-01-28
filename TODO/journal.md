



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