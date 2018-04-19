---
title: 'aws部署golang_beego_nginx'
tags:
- golang
- linux
---


### 通过ssh文件上传到服务器

```
scp -i /Users/liuwei/.ssh/aws.pem -C -r /Users/liuwei/golang/src/web ubuntu@ec2-54-191-9-26.us-west-2.compute.amazonaws.com:/home/ubuntu
```

aws.pem chmod 400

scp  -C 加一个可能会更快

### 发行部署

Go 语言的应用最后编译之后是一个二进制文件，你只需要 copy 这个应用到服务器上，运行起来就行。beego 由于带有几个静态文件、配置文件、模板文件三个目录，所以用户部署的时候需要同时 copy 这三个目录到相应的部署应用之下，下面以我实际的应用部署为例：

<!-- more -->
```
$ mkdir /opt/app/beepkg
$ cp beepkg /opt/app/beepkg
$ cp -fr views /opt/app/beepkg
$ cp -fr static /opt/app/beepkg
$ cp -fr conf /opt/app/beepkg

```
这样在 /opt/app/beepkg 目录下面就会显示如下的目录结构：

```
.
├── conf
│   ├── app.conf
├── static
│   ├── css
│   ├── img
│   └── js
└── views
    └── index.tpl
├── beepkg

```
这样我们就已经把我们需要的应用搬到服务器了，那么接下来就可以开始部署了。

就是一共上传3个文件夹和1个可执行文件

### 独立部署
在 linux 下面部署，我们可以利用 nohup 命令，把应用部署在后端，如下所示：

```
nohup ./beepkg &
```
这样你的应用就跑在了 Linux 系统的守护进程





### Supervisord 管理
Supervisord 是用 Python 实现的一款非常实用的进程管理工具，supervisord 还要求管理的程序是非 daemon 程序，supervisord 会帮你把它转成 daemon 程序，因此如果用 supervisord 来管理 nginx 的话，必须在 nginx 的配置文件里添加一行设置 daemon off 让 nginx 以非 daemon 方式启动。

1. 安装 setuptools

```
wget http://pypi.python.org/packages/2.7/s/setuptools/setuptools-0.6c11-py2.7.egg
sh setuptools-0.6c11-py2.7.egg
easy_install supervisor
echo_supervisord_conf >/etc/supervisord.conf
mkdir /etc/supervisord.conf.d
```

2. 修改配置 /etc/supervisord.conf

```
[include]
files = /etc/supervisord.conf.d/*.conf
```

3. 新建管理的应用
cd /etc/supervisord.conf.d
vim beepkg.conf
配置文件：

```
[program:beepkg]
directory = /opt/app/beepkg
command = /opt/app/beepkg/beepkg
autostart = true
startsecs = 5
user = root
redirect_stderr = true
stdout_logfile = /var/log/supervisord/beepkg.log
```

### supervisord 管理

Supervisord 安装完成后有两个可用的命令行 supervisord 和 supervisorctl，命令使用解释如下：

  ● supervisord，初始启动 Supervisord，启动、管理配置中设置的进程。
  
  ● supervisorctl stop programxxx，停止某一个进程(programxxx)，programxxx 为 [program:beepkg] 里配置的值，这个示例就是 beepkg。
  
  ● supervisorctl start programxxx，启动某个进程
  
  ● supervisorctl restart programxxx，重启某个进程
  
  ● supervisorctl stop groupworker: ，重启所有属于名为 groupworker 这个分组的进程(start,restart 同理)
  
  ● supervisorctl stop all，停止全部进程，注：start、restart、stop 都不会载入最新的配置文件。
  
  ● supervisorctl reload，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
  
  ● supervisorctl update，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。
  
注意：显示用 stop 停止掉的进程，用 reload 或者 update 都不会自动重启。


> 自己的配置

```
web.conf

[program:web]
directory = /home/ubuntu/web
command = /home/ubuntu/web/web
autostart = true
startsecs = 5
user = root
redirect_stderr = true
stdout_logfile = /var/log/supervisord/web.log
```


sudo supervisord //启动
sudo supervisorctl stop web //结束



### nginx部署

1 安装nginx 

```
sudo apt-get install nginx
```

Ubuntu安装之后的文件结构大致为：

  ● 所有的配置文件都在/etc/nginx下，并且每个虚拟主机已经安排在了/etc/nginx/sites-available下
  
  ● 程序文件在/usr/sbin/nginx
  
  ● 日志放在了/var/log/nginx中
  
  ● 并已经在/etc/init.d/下创建了启动脚本nginx
  
  ● 默认的虚拟主机的目录设置在了/var/www/nginx-default (有的版本 默认的虚拟主机的目录设置在了/var/www, 请参考/etc/nginx/sites-available里的配置)

2 启动nginx

```
sudo /etc/init.d/nginx start
```
直接访问ip http://54.191.9.26/ 可以看到nginx安装成功


3 处理golang

Go 是一个独立的 HTTP 服务器，但是我们有些时候为了 nginx 可以帮我做很多工作，例如访问日志，cc 攻击，静态服务等，nginx 已经做的很成熟了，Go 只要专注于业务逻辑和功能就好，所以通过 nginx 配置代理就可以实现多应用同时部署，如下就是典型的两个应用共享 80 端口，通过不同的域名访问，反向代理到不同的应用。

```
server {
    listen       80;
    server_name  .a.com;

    charset utf-8;
    access_log  /home/a.com.access.log;

    location /(css|js|fonts|img)/ {
        access_log off;
        expires 1d;

        root "/path/to/app_a/static";
        try_files $uri @backend;
    }

    location / {
        try_files /_not_exists_ @backend;
    }

    location @backend {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;

        proxy_pass http://127.0.0.1:8080;
    }
}

server {
    listen       80;
    server_name  .b.com;

    charset utf-8;
    access_log  /home/b.com.access.log  main;

    location /(css|js|fonts|img)/ {
        access_log off;
        expires 1d;

        root "/path/to/app_b/static";
        try_files $uri @backend;
    }

    location / {
        try_files /_not_exists_ @backend;
    }

    location @backend {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host            $http_host;

        proxy_pass http://127.0.0.1:8081;
    }
}
```

> 自己的配置

sudo vi /etc/nginx/sites-available/default

把 default 注释掉

```
server {
        listen       80;
        server_name  .xuanyueting.top;

        charset utf-8;
        access_log  /home/ubuntu/web/xuanyueting.log;

        location /(css|js|fonts|img)/ {
                access_log off;
                expires 1d;

                root "/home/ubuntu/web/static";
                try_files $uri @backend;
        }
        location / {
                try_files /_not_exists_ @backend;
        }

        location @backend {
                proxy_set_header x-forwarded-for $remote_addr;
                proxy_set_header host            $http_host;

                proxy_pass http://127.0.0.1:8080;
        }
}
```


