---
title: "nginx常用功能和配置详解"
date: 2019-06-22 23:54:46
tags:
- nginx
- linux
---



Nginx功能丰富，可作为HTTP服务器，也可作为反向代理服务器，邮件服务器。支持FastCGI、SSL、Virtual Host、URL Rewrite、Gzip等功能。并且支持很多第三方的模块扩展。

<!-- more -->

# 1. nginx 常用功能说明



### 1.1 反向代理

下面一张图是对正向代理与反响代理的对比


![1](nginx_config/1.jpg)

Nginx在做反向代理时，提供性能稳定，并且能够提供配置灵活的转发功能。Nginx可以根据不同的正则匹配，采取不同的转发策略，比如图片文件结尾的走文件服务器，动态页面走web服务器，只要你正则写的没问题，又有相对应的服务器解决方案，你就可以随心所欲的玩。并且Nginx对返回结果进行错误页跳转，异常判断等。如果被分发的服务器存在异常，他可以将请求重新转发给另外一台服务器，然后自动去除异常服务器。



### 1.2 负载均衡

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。

内置策略为轮询，加权轮询，Ip hash。扩展策略，就天马行空，只有你想不到的没有他做不到的啦，你可以参照所有的负载均衡算法，给他一一找出来做下实现。



看下面的图理解三种负载均衡算法的实现

![1](nginx_config/2.jpg)



Ip hash算法，对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。

![1](nginx_config/3.jpg)



### 1.3 web缓存

nginx可以对不同的文件做不同的缓存处理，配置灵活，并且支持FastCGI_Cache，主要用于对FastCGI的动态程序进行缓存。配合着第三方的ngx_cache_purge，对制定的URL缓存内容可以的进行增删管理。





# 2. nginx配置文件结构



先放一个默认配置 /etc/nginx/nginx.conf

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```



### 2.1 配置文件结构

+ main(全局块)
+ events (nginx工作模式)

+ http(http设置)1个
  + server(主机设置)多个
    + location(URL匹配)多个
  + upstream(负载均衡服务器设置)1个



1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。

5、location块：配置请求的路由，以及各种页面的处理情况。

6、upstream块：负责负载均衡



```nginx
########### 每个指令必须有分号结束。#################

#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址

error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别分别为：debug|info|notice|warn|error|crit|alert|emerg


events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}


http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
  
 
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
  
    error_page 404 https://www.baidu.com; #错误页
    
  	server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        
    		location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        }
    }
}
```



+ `$remote_addr` 与`$http_x_forwarded_for` 用以记录客户端的ip地址； 
+ `$remote_user` 用来记录客户端用户名称； 
+ `$time_local` 用来记录访问时间与时区；
+ `$request` 用来记录请求的url与http协议；
+ `$status` 用来记录请求状态；成功是200， 
+ `body_bytes_sent` 记录发送给客户端文件主体内容大小；
+ `$http_referer` 用来记录从那个页面链接访问过来的； 8.$http_user_agent ：记录客户端浏览器的相关信息；

+ 惊群现象：一个网路连接到来，多个睡眠的进程被同时叫醒，但只有一个进程能获得链接，这样会影响系统性能。

+ 每个指令必须有分号结束。



# 3. 详细配置说明



### 3.1 main模块

```nginx
user nobody nobody;
worker_processes 2;
error_log  /usr/local/var/log/nginx/error.log  notice;
pid        /usr/local/var/run/nginx/nginx.pid;
worker_rlimit_nofile 1024;
```



+ user 来指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。

+ worker_processes来指定了Nginx要开启的子进程数。每个Nginx进程平均耗费10M~12M内存。根据经验，一般指定1个进程就足够了，如果是多核CPU，建议指定和CPU的数量一样的进程数即可。我这里写2，那么就会开启2个子进程，总共3个进程。

+ error_log用来定义全局错误日志文件。日志输出级别有debug、info、notice、warn、error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。

+ pid用来指定进程id的存储文件位置。

+ worker_rlimit_nofile用于指定一个nginx进程可以打开的最多文件描述符数目，这里是65535，需要使用命令“ulimit -n 65535”来设置。

  

### 3.2 events 模块

events模块来用指定nginx的工作模式和工作模式及连接数上限

```nginx
events {
     use kqueue; #mac平台
     worker_connections  1024;
}
```



+ use用来指定Nginx的工作模式。Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中，因为Mac基于BSD,所以Mac也得用这个模式，对于Linux系统，epoll工作模式是首选。

+ worker_connections用于定义Nginx每个进程的最大连接数，即接收前端的最大请求数，默认是1024。最大客户端连接数由worker_processes和worker_connections决定
+ 即Max_clients=worker_processes*worker_connections，在作为反向代理时，Max_clients变为：Max_clients = worker_processes * worker_connections/4。 

+ 进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后worker_connections的设置才能生效。



### 3.3 http 模块

http模块可以说是最核心的模块了，它负责HTTP服务器相关属性的配置，它里面的server和upstream子模块，至关重要。

```nginx
http {
     include       mime.types;
     default_type  application/octet-stream;
     log_format  main  'remote_addr - remote_user [time_local] "request" '
                       'status body_bytes_sent "$http_referer" '
                       '"http_user_agent" "http_x_forwarded_for"';
     access_log  /usr/local/var/log/nginx/access.log  main;
     sendfile        on;
     tcp_nopush      on;
     tcp_nodelay     on;
     keepalive_timeout  10;
     #gzip  on;
     
     upstream myproject {
         .....
     }
     
  	 server {
         ....
     }
}
```



+ include 用来设定文件的mime类型,类型在配置文件目录下的mime.type文件定义，来告诉nginx来识别文件类型。

+ default_type设定了默认的类型为二进制流，也就是当文件类型未定义时使用这种方式。

+ log_format用于设置日志的格式，和记录哪些参数，这里设置为main，刚好用于access_log来记录这种类型。

  main的类型日志如下：也可以增删部分参数。

  127.0.0.1 - - [21/Apr/2015:18:09:54 +0800] "GET /index.php HTTP/1.1" 200 87151 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36" 

+ access_log 用来纪录每次的访问日志的文件地址，后面的main是日志的格式样式，对应于log_format的main。

+ sendfile参数用于开启高效文件传输模式。
+ 将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞。

+ keepalive_timeout设置客户端连接保持活动的超时时间。在超过这个时间之后，服务器会关闭该连接。



### 3.4 server 模块 (http的子模块, 虚拟主机, 最重要)



sever 模块是http的子模块，它用来定一个虚拟主机。



我们看一下一个简单的server 是如何做的？

```nginx
server {
         listen       8080;
         server_name  test.liuvv.com;
         root   /home/levonfly/www;         # 全局定义，如果都是这一个目录，这样定义最简单。
         index  index.php index.html index.htm; 
         charset utf-8;
         access_log  usr/local/var/log/host.access.log  main;
         aerror_log  usr/local/var/log/host.error.log  error;
         ....
}

```



+ server标志定义虚拟主机开始。 

+ listen用于指定虚拟主机的服务端口。 

+ server_name用来指定IP地址或者域名，多个域名之间用空格分开。 

+ root 表示在这整个server虚拟主机内，全部的root web根目录。注意要和locate {}下面定义的区分开来。 

+ index 全局定义访问的默认首页地址。注意要和locate {}下面定义的区分开来。 

+ charset用于设置网页的默认编码格式。 

+ access_log用来指定此虚拟主机的访问日志存放路径，最后的main用于指定访问日志的输出格式。



### 3.5 location 模块(server的子模块)

location模块是nginx中用的最多的，也是最重要的模块了，什么负载均衡啊、反向代理啊、虚拟域名啊都与它相关。

location 根据它字面意思就知道是来定位的，定位URL，解析URL，所以，它也提供了强大的正则匹配功能，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤处理。



我们先来看这个，设定默认首页和虚拟机目录。

```nginx
location / {
	root   /home/levonfly//www;
	index  index.php index.html index.htm;
}
```

+ location /表示匹配访问根目录。

+ root指令用于指定访问根目录时，虚拟主机的web目录，这个目录可以是相对路径（相对路径是相对于nginx的安装目录）。也可以是绝对路径。

+ index用于设定我们只输入域名后访问的默认首页地址，有个先后顺序：index.php index.html index.htm，如果没有开启目录浏览权限，又找不到这些默认首页，就会报403错误。



### 3.6 upstream 模块(http子模块)

upstream 模块负债负载均衡模块，通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。我先学习怎么用，具体的使用实例以后再说。

```nginx
upstream liuvv.com{
     ip_hash;
     server 192.168.12.1:80;
     server 192.168.12.2:80 down;
     server 192.168.12.3:8080  max_fails=3  fail_timeout=20s;
     server 192.168.12.4:8080;
}
```

在上面的例子中，通过upstream指令指定了一个负载均衡器的名称liuvv.com。这个名称可以任意指定，在后面需要的地方直接调用即可。



里面是ip_hash这是其中的一种负载均衡调度算法，下面会着重介绍。紧接着就是各种服务器了。用server关键字表识，后面接ip。

Nginx的负载均衡模块目前支持4种调度算法:

1. weight 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。weight。指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。
2. ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。
3. fair。比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。
4. url_hash。按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包。



在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

- down，表示当前的server暂时不参与负载均衡。
- backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
- max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
- fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

**注意** 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。





# 4. 参考资料



https://www.zybuluo.com/phper/note/89391 nginx的配置、虚拟主机、负载均衡和反向代理

https://www.zybuluo.com/phper/note/90310  2

https://www.zybuluo.com/phper/note/133244  3



https://juejin.im/post/5aa7704c6fb9a028bb18a993 //Nginx 基本配置详解

https://my.oschina.net/duxuefeng/blog/34880 //nginx配置详解

https://www.jianshu.com/p/a7c86efe1987    //nginx配置文件详解中文版

http://www.nginx.cn/591.html  //nginx配置入门



https://jkzhao.github.io/2018/01/23/Nginx%E9%85%8D%E7%BD%AE%E8%AF%A6%E8%A7%A3%E5%8F%8A%E4%BC%98%E5%8C%96   //Nginx配置详解及优化/

https://www.kancloud.cn/curder/nginx/96672 //nginx学习笔记

 https://nginxconfig.io/ //在线生成
