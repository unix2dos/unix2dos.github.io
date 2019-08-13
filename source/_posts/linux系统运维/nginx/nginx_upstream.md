---
title: nginx通过upstream实现负载均衡
tags:
  - nginx
abbrlink: 4c38afcc
categories:
  - linux系统运维
  - nginx
date: 2019-07-06 11:45:46
---



### 1. nginx upstream 负载均衡

upstream 模块负债负载均衡模块，如果Nginx没有仅仅只能代理一台服务器的话，那它也不可能像今天这么火，Nginx可以配置代理多台服务器，当一台服务器宕机之后，仍能保持系统可用。具体配置过程如下：

 在http节点下，添加upstream节点。

```nginx
upstream levonfly { 
      server 10.0.6.108:7080; 
      server 10.0.0.85:8980; 
}
```

将server节点下的location节点中的proxy_pass配置为：http:// + upstream名称，即"http://levonfly".

```nginx
location / { 
            root  html; 
            index  index.html index.htm; 
            proxy_pass http://levonfly; 
}
```

现在负载均衡初步完成了。upstream按照轮询（默认）方式进行负载，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。虽然这种方式简便、成本低廉。
<!-- more -->


### 2. upstream 负载均衡的模式

nginx的upstream支持5种分配方式，下面将会详细介绍，其中，前三种为Nginx原生支持的分配方式，后两种为第三方支持的分配方式：

##### 2.1 轮询(默认)

轮询是upstream的默认分配方式，即每个请求按照时间顺序轮流分配到不同的后端服务器，如果某个后端服务器down掉后，能自动剔除。

```nginx
upstream backend {
  server 192.168.1.101:8888;
  server 192.168.1.102:8888;
  server 192.168.1.103:8888;
}
```

##### 2.2 weight        

轮询的加强版，即可以指定轮询比率，weight和访问几率成正比，主要应用于后端服务器异质的场景下。

```nginx
upstream backend {
  server 192.168.1.101 weight=1;
  server 192.168.1.102 weight=2;
  server 192.168.1.103 weight=3; # 是101访问率的3倍
}
```

##### 2.3 ip_hash        

每个请求按照访问ip（即nginx的前置服务器或者客户端IP）的hash结果分配，这样每个访客会固定访问一个后端服务器，可以解决session一致问题。

```nginx
upstream backend {
  ip_hash;
  server 192.168.1.101:7777;
  server 192.168.1.102:8888;
  server 192.168.1.103:9999;
}
```

##### 2.4 fair(第三方) 

fair顾名思义，公平地按照后端服务器的响应时间（rt）来分配请求，响应时间短即rt小的后端服务器优先分配请求。

```nginx
upstream backend {
  server 192.168.1.101;
  server 192.168.1.102;
  server 192.168.1.103;
  fair;
}
```

##### 2.5 url_hash(第三方)

与ip_hash类似，但是按照访问url的hash结果来分配请求，使得每个url定向到同一个后端服务器，主要应用于后端服务器为缓存时的场景下。

```nginx
upstream backend {
  server 192.168.1.101;
  server 192.168.1.102;
  server 192.168.1.103;
  hash $request_uri;
  hash_method crc32;
}
```

注意：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。



### 3. upstream 其他参数

upstream中server指令语法如下：`server address [parameters]` ,  关键字server必选, address也必选，可以是主机名、域名、ip或unix socket，也可以指定端口号。

upstream还可以为每个设备设置状态值，这些状态值的含义分别如下：

+ down 表示单前的server暂时不参与负载.

+ weight 默认为1.    weight越大，负载的权重就越大。

+ max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.

+ fail_timeout : max_fails次失败后，暂停的时间。

+ backup： 表示当前server是备用服务器，只有其它非backup后端服务器都挂掉了或者很忙才会分配到请求。

```nginx
upstream bakend{ #定义负载均衡设备的Ip及设备状态 
      ip_hash; 
      server 10.0.0.11:9090 down; 
      server 10.0.0.11:8080 weight=2; 
      server 10.0.0.11:6060; 
      server 10.0.0.11:7070 backup; 
}
```

##### 3.1 注意:

max_fails和fail_timeout一般会关联使用，如果某台server在fail_timeout时间内出现了max_fails次连接失败，那么Nginx会认为其已经挂掉了，从而在fail_timeout时间内不再去请求它，fail_timeout默认是10s，max_fails默认是1，即默认情况是只要发生错误就认为服务器挂掉了，如果将max_fails设置为0，则表示取消这项检查。



### 4. 实战配置:

````nginx
upstream levonfly {
    server 127.0.0.1:13050;
}

upstream testing-levonfly {
    server 172.25.61.25:13050;
}


server {
    server_name levonfly.com;
    listen 443 ssl http2 ;
    access_log /var/log/nginx/cistern_access_log;
    error_log /var/log/nginx/cistern_error_log notice;
    ssl_certificate /etc/nginx/certs/STAR.levonfly.com.crt;
    ssl_certificate_key /etc/nginx/certs/STAR.levonfly.com.key;
    proxy_set_header       X-Real-IP $remote_addr;
    proxy_set_header       X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header       X-Forwarded-Proto $scheme;
    proxy_set_header       X-Scheme $scheme;
    proxy_set_header       Host $http_host;
    proxy_redirect         off;
    proxy_intercept_errors on;

    location / {
        proxy_pass http://levonfly;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
    }
}

server {
    server_name test.levonfly.com;
    listen 443 ssl http2 ;
    access_log /var/log/nginx/cistern_access_log;
    error_log /var/log/nginx/cistern_error_log notice;
    ssl_certificate /etc/nginx/certs/STAR.levonfly.com.crt;
    ssl_certificate_key /etc/nginx/certs/STAR.levonfly.com.key;
    proxy_set_header       X-Real-IP $remote_addr;
    proxy_set_header       X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header       X-Forwarded-Proto $scheme;
    proxy_set_header       X-Scheme $scheme;
    proxy_set_header       Host $http_host;
    proxy_redirect         off;
    proxy_intercept_errors on;

    location / {
        proxy_pass http://testing-levonfly;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
    }
}
````

##### 4.1 遇到的问题:

 如果不生效, 注意下 upstream 的缩进会造成问题



### 5. 参考资料

+ http://www.linuxe.cn/post-182.html

+ https://www.cnblogs.com/wzjhoutai/p/6932007.html

