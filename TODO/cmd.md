### 设置linux时间和时区

```bash
date 
date -s "2015-10-25 15:00:00" #修改时间


#设置时区
tzselect  #命令只告诉你选择的时区的写法，并不会生效。
在.bashrc添加  export TZ='Asia/Shanghai'
```



### 查看linux系统版本

```bash
lsb_release -a 

uname -srm  

cat /etc/os-release

cat /proc/version

cat /etc/issue
```



### 域名查询IP

```bash
host hostname [server]

[server]：使用不是由/etc/resolv.conf文件定义的DNS服务器IP来查询某台主机的IP。

host www.baidu.com
host www.baidu.com 8.8.8.8
```



