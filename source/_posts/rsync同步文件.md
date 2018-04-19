---
title: 'rsync同步文件'
tags: linux
---


参考链接: http://blog.csdn.net/zpf336/article/details/51659666


### 把本地文件同步到远程服务器
```
rsync -avz '-e ssh -i /Users/liuwei/.ssh/aws.pem' /Users/liuwei/golang/src/web --progress ubuntu@54.191.9.26:/home/ubuntu
```

<!-- more -->
### client

```
rsync -vzrtopg --progress  ubuntu@54.191.9.26::ftp .   //同步服务器的文件到当前目录
```

### server

```
[ftp]

        comment = public archive
        path = /home/ubuntu/rsync
        use chroot = yes
#       max connections=10
        lock file = /var/lock/rsyncd
# the default for read only is yes...
        read only = yes
        list = yes
        uid = nobody
        gid = nogroup
#       exclude =
#       exclude from =
#       include =
#       include from =
        auth users =  ubuntu
        secrets file = /etc/rsyncd.secrets
        strict modes = yes
#       hosts allow =
#       hosts deny =
        ignore errors = no
        ignore nonreadable = yes
        transfer logging = no
#       log format = %t: host %h (%a) %o %f (%l bytes). Total %b bytes.
        timeout = 600
        refuse options = checksum dry-run
        dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz

```



### 在本地机器上对两个目录同步

```
rsync -zvr filename1 filename2
```
上述代码是将filename1中的文件与filename2中的文件同步

### 使用rsync –a 同步保留时间按标记

```
rsync -azv filename1 filename2  
```

使用上述命令，将filename2中新同步的文件的时间与filename1中的创建的时间相同，它保留符号链接、权限、时间标记、用户名及组名相同。

### 将远程服务器的文件同步到本地

```
rsync -avz ubuntu@192.168.0.1:/home/ubuntu/filename2 filename1 
```

上述命令是将远程192.168.0.1的主机上filename2同步到本地的filename1。
注意：如果远程主机的端口不是默认的22端口，假如是4000端口，上述的命令修改为，

```
rsync -avz '-e ssh -p 4000' ubuntu@192.168.0.1:/home/ubuntu/filename2 filename1 
```

### 从本地同步文件到远程服务器

```
rsync -avz filename1 ubuntu@192.168.0.1:/home/ubuntu/filename2  
```
上述命令是将本地的filename1同步到远程192.168.0.1的主机上。
同理如果端口不是22，使用以下命令

```
rsync -avz '-e ssh -p 4000' filename1 ubuntu@192.168.0.1:/home/ubuntu/filename2  
```
