---
title: linux开启ftp服务和golang实现ftp_server_client
tags: ["golang"]
abbrlink: d43abcbd
categories:
  - 1-编程语言
  - golang
date: 2018-05-30 22:42:26
---



### linux 安装 ftp 服务

1 . 安装ftp

```
sudo apt-get install vsftpd
```

2. 修改配置  sudo vi /etc/vsftpd.con

```
local_root=/home/ftpuser
write_enable=YES
anon_mkdir_write_enable=YES
```


3. 添加ftp用户

```
mkdir /home/ftpuser
sudo useradd -d /root/workspace -M ftpuser
sudo passwd ftpuser
```

4. 调整文件夹权限

```
chown ftpuser:ftpuser  /home/ftpuser/
sudo chmod a-w  /home/ftpuser 
```

5. 修改pam.d/vsftpd

```
sudo vi /etc/pam.d/vsftpd
#auth    required pam_shells.so //注释掉这一行
sudo service vsftpd restart
```

6. 连接

```
ftp://207.246.80.69  //通过浏览器访问

mac 可以下载 filezilla 客户端进行连接
```

<!-- more -->

## golang 实现 ftp-server ftp-client

### server

https://github.com/fclairamb/ftpserver 

### client

https://github.com/secsy/goftp

https://github.com/jlaffaye/ftp

### io progress

https://github.com/mitchellh/ioprogress

#### 注意事项:

+ 显示进度的时候要确定总的size

+ 在显示进度的时候要注意设置断点续传的进度

+ 列出file的名字

