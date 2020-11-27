---
title: php8.0安装和php-fpm
tags:
  - php
categories:
  - 1-编程语言
  - php
abbrlink: ad42ac48
date: 2020-11-27 00:00:01
---

# 1. 安装

### 1.1  下载

https://www.php.net/downloads  下载php-8.0.0.tar.gz

<!-- more -->

### 1.2 安装

```bash
tar zxvf php-8.0.0.tar.gz


sudo apt install -y pkg-config build-essential autoconf bison re2c libxml2-dev libsqlite3-dev

./configure --prefix=/usr/local/php  --enable-fpm

make && make  install


# 最后安装如下
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20200930/
Installing PHP CLI binary:        /usr/local/php/bin/
Installing PHP CLI man page:      /usr/local/php/php/man/man1/
Installing PHP FPM binary:        /usr/local/php/sbin/
Installing PHP FPM defconfig:     /usr/local/php/etc/
Installing PHP FPM man page:      /usr/local/php/php/man/man8/
Installing PHP FPM status page:   /usr/local/php/php/php/fpm/
Installing phpdbg binary:         /usr/local/php/bin/
Installing phpdbg man page:       /usr/local/php/php/man/man1/
Installing PHP CGI binary:        /usr/local/php/bin/
Installing PHP CGI man page:      /usr/local/php/php/man/man1/
Installing build environment:     /usr/local/php/lib/php/build/
Installing header files:          /usr/local/php/include/php/
Installing helper programs:       /usr/local/php/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php/php/man/man1/
  page: phpize.1
  page: php-config.1
/root/workspace/php-8.0.0/build/shtool install -c ext/phar/phar.phar /usr/local/php/bin/phar.phar
ln -s -f phar.phar /usr/local/php/bin/phar
Installing PDO headers:           /usr/local/php/include/php/ext/pdo/


# 查看版本
/usr/local/php/bin/php -v 
```



### 1.3 启动 php-fpm

```bash
cd /usr/local/php/etc
cp php-fpm.conf.default  php-fpm.conf
vi php-fpm.conf   
去掉# pid = run/php-fpm.pid 前面的注释


cd /usr/local/php/etc/php-fpm.d
cp www.conf.default  www.conf


# 测试
/usr/local/php/sbin/php-fpm -t

# 启动
/usr/local/php/sbin/php-fpm  # 报错 cannot get gid for group 'nobody'
groupadd nobody # 增加组即可
```



### 1.4 配置

+ nginx 配置

` vi /etc/nginx/sites-enabled/php.conf`

```nginx
server {
        listen          80; 
        server_name  php.liuvv.com;
        root   /var/www/html/php;
        index  index.html index.htm index.php;

        location ~ \.php {
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
#               fastcgi_split_path_info ^(.+\.php)(/.*)$;
#               fastcgi_param PATH_INFO $fastcgi_path_info;
                include        fastcgi_params;
        }   
}
```

+ 增加 php 文件

```bash
 mkdir /var/www/html/php
 vi /var/www/html/php/index.php
 <? phpinfo();
```



# 2. fastcgi和php-fpm

### 2.1 CGI协议

CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。

CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。

当Nginx收到`/index.php`这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。



### 2.2 FastCGI协议

FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次（这是CGI最为人诟病的fork-and-execute 模式）。它还支持分布式的运算，即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。



FastCGI的特点是会在一个进程中依次完成多个请求，以达到提高效率的目的，大多数FastCGI实现都会维护一个进程池。



### 2.3 PHP-CGI解释器

PHP-CGI是PHP自带的FastCGI管理器。

PHP-CGI的不足：

1. php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
2. 直接杀死php-cgi进程，php就不能运行了。

PHP-CGI只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理。所以就出现了一些能够调度 php-cgi 进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。同样，PHP-FPM也是用于调度管理PHP解析器php-cgi的管理程序。



### 2.4 PHP-FPM调度PHP-CGI

PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）

所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。PHP-FPM也是用于调度管理PHP解析器php-cgi的管理程序。



### 2.5 问题回答

+ fastcgi是一个协议，php-fpm实现了这个协议； 

  对。

+ 有的说，php-fpm是fastcgi进程的管理器，用来管理fastcgi进程的； 

  对。

  php-fpm的管理对象是php-cgi。但不能说php-fpm是fastcgi进程的管理器，fastcgi是个协议，似乎没有这么个进程存在，就算存在php-fpm也管理不了他（至少目前是）。

+ 有的说，php-fpm是php内核的一个补丁; 

  以前是对的。

  因为最开始的时候php-fpm没有包含在PHP内核里面，要使用这个功能，需要找到与源码版本相同的php-fpm对内核打补丁，然后再编译。后来PHP内核集成了PHP-FPM之后就方便多了，使用--enalbe-fpm这个编译参数即可。

+ 有的说，修改了php.ini配置文件后，没办法平滑重启，所以就诞生了php-fpm； 

  是的。
  修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。

+ 还有的说php-cgi是PHP自带的FastCGI管理器，那这样的话干吗又弄个php-fpm出来；

  php-cgi与php-fpm一样，也是一个fastcgi进程管理器

  php-cgi的问题在于 1、php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启 2、直接杀死php-cgi进程,php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题,守护进程会平滑从新生成新的子进程。） 针对php-cgi的不足，php-fpm应运而生。



# 3. 参考资料

+ https://www.jianshu.com/p/7627c794b272

+ https://segmentfault.com/q/1010000000256516

