\+ 设置时间

date

date -s "2015-10-25 15:00:00"



\+ 设置时区

tzselect命令只告诉你选择的时区的写法，并不会生效。

在.bashrc添加  export TZ='Asia/Shanghai'





\+ 查看系统版本

lsb_release -a 

uname -srm  

cat /etc/os-release

cat /proc/version

cat /etc/issue