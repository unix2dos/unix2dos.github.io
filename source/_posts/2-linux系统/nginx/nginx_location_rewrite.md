---
title: nginx的location用法和rewrite规则及proxy_pass模块
tags:
  - nginx
  - linux
abbrlink: 51e59d76
categories:
  - 2-linux系统
  - nginx
date: 2019-07-11 23:54:46
---

# 0. 前言

uri是url中除去协议和域名及参数后, 剩下的部分.

比如请求的url为: http://www.liuvv.com/test/index.php?page=1, 则uri 为 `/test/index.php`

<!-- more -->

# 1. location指令

### 1.1 location匹配uri的规则:

```
location [ = | ~ | ~* | ^~ ] uri { ... }
```

+ `=`  精确匹配, uri要完全一样
+ `^~`  这个是以某个开头, 不是正则匹配
+ `~`  区分大小写正则匹配
+ `~*` 不区分大小写正则匹配



### 1.2 location匹配uri的优先级:

1. 首先先检查使用前缀字符定义的location，选择最长匹配的项并记录下来。

2. 如果找到了精确匹配的location，也就是使用了`=`修饰符的location，结束查找，使用它的配置。

3. 如果`^~`修饰符先匹配到最长的前缀字符串, 则不检查正则。

4. 然后按顺序查找使用正则定义的location，如果匹配则停止查找，使用它定义的配置。

5. 如果没有匹配的正则location，则使用前面记录的最长匹配前缀字符location。

   

> 基于以上的匹配过程，我们可以得到以下启示：

1. 使用正则定义的location在配置文件中出现的顺序很重要。因为找到第一个匹配的正则后，查找就停止了，后面定义的正则就是再匹配也没有机会了。
2. 使用精确匹配可以提高查找的速度。例如经常请求`/`的话，可以使用`=`来定义location。
3. 优先级 `=`  >  `^~`  >  正则



### 1.3 location 测试

```nginx
location = / {
  return 502 "规则A\n";
}
location = /login {
  return 502 "规则B\n";
}
location ^~ /static/ {
  return 502 "规则C\n";
}
location ^~ /static/files {
  return 502 "规则D\n";
}
location ~ \.(gif|jpg|png|js|css)$ {
  return 502 "规则E\n";
}
location ~* \.PNG$ {
  return 502 "规则F\n";
}
location /img {
  return 502 "规则G\n";
}
location / {
  return 502 "规则H\n";
}
```

测试结果:

```bash
levonfly@hk:~$ curl test.liuvv.com # 因为是=
规则A
levonfly@hk:~$ curl test.liuvv.com/ # 因为是=
规则A
levonfly@hk:~$ curl test.liuvv.com/login # 因为是=
规则B
levonfly@hk:~$ curl test.liuvv.com/login/ # 多了/, 参考下面3.5
规则H
levonfly@hk:~$ curl test.liuvv.com/abc
规则H

levonfly@hk:~$ curl test.liuvv.com/static/a.html 
规则C
levonfly@hk:~$ curl test.liuvv.com/static/files/a.html # 更加精准
规则D

levonfly@hk:~$ curl test.liuvv.com/a.png # 正则精准
规则E
levonfly@hk:~$ curl test.liuvv.com/a.PNG # 正则精准
规则F
levonfly@hk:~$ curl test.liuvv.com/static/a.png # ^~ 优先级更高
规则C

levonfly@hk:~$ curl test.liuvv.com/img/a.gif #正则匹配优先
规则E
levonfly@hk:~$ curl test.liuvv.com/img/a.tiff
规则G
levonfly@hk:~$ curl test.liuvv.com/abc/123/haha #什么都匹配不到, 就到H
规则H
```



### 1.4 location @name的用法

@用来定义一个命名location。主要用于内部重定向，不能用来处理正常的请求。其用法如下：

```nginx
location / {
    try_files $uri $uri/ @custom
}
location @custom {
    # ...do something
}
```

上例中，当尝试访问url找不到对应的文件就重定向到我们自定义的命名location（此处为custom）。

值得注意的是，命名location中不能再嵌套其它的命名location。




### 1.5 root和alias的区别

nginx指定文件路径有两种方式root和alias.

- root

```
[root]
语法：root path
默认值：root html
配置段：http、server、location、if
```

例如:

```nginx
location ^~ /t/ { 
     root /www/root/html/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/t/a.html的文件。

- alias

```
[alias]
语法：alias path
配置段：location
```

例如:

```nginx
location ^~ /t/ { # 特殊的规则是, alias必须以"/" 结束
 alias /www/root/html/new_t/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/new_t/a.html的文件。注意这里是new_t，因为alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录。



### 1.6 nginx显示目录结构

nginx默认是不允许列出整个目录的。如需此功能, 在server或location 段里添加上autoindex on;

```nginx
autoindex_exact_size off;
默认为on，显示出文件的确切大小，单位是bytes。
改为off后，显示出文件的大概大小，单位是kB或者MB或者GB

autoindex_localtime on;
默认为off，显示的文件时间为GMT时间。
改为on后，显示的文件时间为文件的服务器时间
```

可以下面的例子:

```nginx
location ^~ "/upload-preview" {
		alias /tmp/cistern/;
		autoindex on;
		autoindex_localtime on;
}
```



### 1.7 URL尾部的`/`需不需要

关于URL尾部的`/`有三点也需要说明一下。第一点与location配置有关，其他两点无关。

+ location配置中的字符有没有`/`都没有影响(只是location, 不是alias)。也就是说`/user/`和`/user`是一样的。

+ 如果URL结构是`https://domain.com/`的形式，尾部有没有`/`都不会造成重定向。因为浏览器在发起请求的时候，默认加上了`/`。虽然很多浏览器在地址栏里也不会显示`/`。这一点，可以访问[baidu](https://www.baidu.com/)验证一下。

+ 如果URL的结构是`https://domain.com/some-dir/`。尾部如果缺少`/`将导致重定向。因为根据约定，URL尾部的`/`表示目录，没有`/`表示文件。

  所以访问`/some-dir/`时，服务器会自动去该目录下找对应的默认文件。如果访问`/some-dir`的话，服务器会先去找`some-dir`文件，找不到的话会将`some-dir`当成目录，重定向到`/some-dir/`，去该目录下找默认文件。



# 2. rewirte规则


### 2.1 return指令

return指令写在server和location里面

```nginx
return code [text];
return code URL;
return URL;
```

我们来看下面这个例子

```nginx
 return 301 $scheme://www.baidu.com$request_uri; 
```

return 指令告诉 nginx 停止处理请求, 直接返回301代码和指定重写过的URL到客户端. $scheme是指协议(http),$request_uri指包含参数的完整URI 



对于 3xx 系列响应码, url参数就是重写的url

```nginx
return (301 | 302 | 303 | 307) url;
```

对于其他响应吗, 可以出现一个字符串

```nginx
return (1xx | 2xx | 4xx | 5xx)["text"]
```

例如:

```nginx
return 401 "Access denied because token is expired or invalid";
```



### 2.2 rewrite指令

rewrite指令写在server和location里面, 规则会改变部分或整个用户的URL.

```nginx
rewrite regex URL [flag]
```

1. regex 正则表达式

2. flag

   + last

     停止当前这个请求，并根据rewrite匹配的规则重新发起一个请求。新请求又从第一阶段开始执行, 找寻新的location

   + break

     break并不会重新发起一个请求，只是跳过当前的rewrite阶段，并执行本请求后续的执行阶段, 在同一个location里处理

   + redirect

     返回包含302的临时重定向

   + permanent

     返回包含301的永久重定向

3. rewrite只能返回301或302, 如果有其他,需要后面加上return, 例如:

```nginx
rewrite ^(/download/.*)/media/(\w+)\.?.*$ $1/mp3/$2.mp3 last;
rewrite ^(/download/.*)/audio/(\w+)\.?.*$ $1/mp3/$2.ra  last;
return  403;
```

+ 匹配/download开头的URL,  然后替换相关路径
+ `/download/cdn-west/media/file1` -> `/download/cdn-west/mp3/file1.mp3`

+ 如果匹配不上, 将返回客户端403



### 2.3 try_files

try_files指令写在server和location里面.

```nginx
try_files file ... uri 或 try_files file ... = code
```

try_files 指令的参数是一个或多个文件或目录的列表, 以及后面的uri参数. nginx会按照顺序检查文件或目录是否存在, 并用找到的第一个文件提供服务. 如果都不存在, 内部重定向到最后的这个uri

例如:

```nginx
location /images/ {
    try_files $uri $uri/ /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

try_files常用的变量:

+ $uri 表示域名以后的部分

+ $args  请求url中 ? 后面的参数 (不包括?本身)

+ $is_args  判断$args是否为空

  ```nginx
  try_files $uri $uri/ /index.php$is_args$args; #这样就能避免多余的?号
  ```
  
+ $query_string 和 $args相同

+ $document_root root指令指定的值

+ $request_filename 请求的文件路径

+ $request_uri 原始请求uri  



我们看个例子:

```nginx
try_files /app/cache/ $uri @fallback; 
index index.php index.html;
```

它将检测$document_root/app/cache/index.php,$document_root/app/cache/index.html 和 $document_root$uri是否存在，如果不存在着内部重定向到@fallback(＠表示配置文件中预定义标记点) 。

你也可以使用一个文件或者状态码(=404)作为最后一个参数，如果是最后一个参数是文件，那么这个文件必须存在。



我们来看一个错误:

```nginx
location ~.*\.(gif|jpg|jpeg|png)$ {
        root /web/wwwroot;
        try_files /static/$uri $uri;
}
```

原意图是访问`http://example.com/test.jpg`时先去检查`/web/wwwroot/static/test.jpg`是否存在，不存在就取`/web/wwwroot/test.jpg`

但由于最后一个参数是一个内部重定向，所以并不会检查`/web/wwwroot/test.jpg`是否存在，只要第一个路径不存在就会重新向然后再进入这个location造成死循环。结果出现500 Internal Server Error

```nginx
location ~.*\.(gif|jpg|jpeg|png)$ {
        root /web/wwwroot;
        try_files /static/$uri $uri 404;
}
```

这样才会先检查`/web/wwwroot/static/test.jpg`是否存在，不存在就取`/web/wwwroot/test.jpg`再不存在则返回404 not found



### 2.4 if指令

if不是系统级的指令, 是和rewrite配合的. if 必须写在server和location里面.

- 变量名:   如果是空字符串或"0"为FALSE
- = 判断相等, != 判断不相等
- ~ 和 ~*(不分区大小写) 将变量与正则匹配, 捕获可以用 $1 到 $9
- !~ 和 !~* 用作不匹配运算符
- 正则含有 } 或 ; 字符需要用引号括起来
- 常用判断指令
  - -f 和 !-f 判断是否存在文件(file)
  - -d 和 !-d 判断是否存在目录(directory)
  - -e 和 !-e 判断是否存在文件或目录(exists)
  - -x 和 !-x 判断文件是否可执行(execute)



例如下面的列子:

```nginx
if ($http_user_agent ~ Chrome) {
    rewrite ^([^/]*)$ /chrome$1 break;
}

if ($request_method = POST){
    return 405;
}

if (-f $request_filename) {
    expires max;
    break;
}
```



# 3. proxy_pass模块

proxy_pass指令是将请求反向代理到URL参数指定的服务器上，URL可以是主机名或者IP地址+端口号的形式，例如：

```
proxy_pass http://proxy_server;
proxy_pass http://192.168.9.2:8000;
proxy_pass https://192.168.9.2:8000;
```

proxy_pass模块基本配置： 
+ proxy_set_header：设置服务器获取用户的主机名或者真实ip地址，以及代理者的真实ip地址。 
+ client_body_buffer_size：用于指定客户端请求主体缓冲区大小，可以理解为先保存到本地再传给用户 
+ proxy_connect_timeout：表示连接服务器的超时时间，即发起tcp握手等候响应的超时时间 
+ proxy_send_time：服务器的数据传回时间，在规定时间内服务器必须传回完所有数据，否则，nginx将断开这个连接 
+ proxy_read_time：设置nginx从代理的后端服务器获取数据的时间，表示连接建立成功后，+ nginx等待服务器的响应时间，其实是nginx已经进入服务器的排队中等候处理的时间。 
+ proxy_buffer_size：设置缓冲区大小，默认该缓冲区大小等于proxy_buffers设置的大小 
+ proxy_buffers：设置缓冲区的数量和大小，nginx从代理的服务器获取响应数据会放置到缓冲区 
+ proxy_busy_buffers_size：用于设置系统很忙时可以使用的proxy_buffers大小，官方推荐大小为proxy_buffers*2 
+ proxy_temp_file_write_size：指定proxy缓存临时文件的大小



### 3.1 proxy_set_header

```bash
语法:    proxy_set_header field value;
默认值:    
proxy_set_header Host $proxy_host; # 注意这个是proxy_host
proxy_set_header Connection close;


上下文:    http, server, location
```

允许重新定义或者添加发往后端服务器的请求头。value可以包含文本、变量或者它们的组合。 当且仅当当前配置级别中没有定义proxy_set_header指令时，会从上面的级别继承配置。默认情况下，只有两个请求头会被重新定义：

```nginx
proxy_set_header Host       $proxy_host;
proxy_set_header Connection close;
```



proxy_set_header也可以自定义参数，如：proxy_set_header test paroxy_test;

如果想要支持下划线的话，需要增加如下配置：`underscores_in_headers on`; 

```bash
语法：underscores_in_headers on|off
默认值：off
使用字段：http, server
是否允许在header的字段中带下划线
```



### 3.2 遇到的问题

> 经过反向代理后，由于在客户端和web服务器之间增加了中间层，因此web服务器无法直接拿到客户端的ip, 通过$remote_addr变量拿到的将是反向代理服务器的ip地址. 如果我们想要在web端获得用户的真实ip，就必须在nginx这里作一个赋值操作，如下：

```nginx
proxy_set_header            X-real-ip $remote_addr;
```

其中这个X-real-ip是一个自定义的变量名，名字可以随意取，这样做完之后，用户的真实ip就被放在X-real-ip这个变量里了，然后，在web端可以这样获取：request.getAttribute("X-real-ip")



>  通常我们会看到有这样一些配置:

```
server {
        server_name liuwei.fhyx.com;

        proxy_set_header       X-Real-IP $remote_addr;
        proxy_set_header       X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header       Host $http_host;
        proxy_redirect         off;
        proxy_http_version     1.1;
        proxy_set_header       Upgrade $http_upgrade;
        proxy_set_header       Connection "upgrade";
        proxy_buffering        off;

        location / {
                proxy_pass     http://127.0.0.1:8000;
        }

        location ~ "/(box|c|bub)/" {
                proxy_pass     http://127.0.0.1:8081;
        }

        location ~ /(a|o2)/ {
                proxy_pass     http://127.0.0.1:3010;
        }

        location ~ "/api/(v\d{1,2})/" {
                proxy_pass     http://127.0.0.1:5010;
        }

}
```

##### proxy_set_header   X-real-ip $remote_addr;

这句话之前已经解释过，有了这句就可以在web服务器端获得用户的真实ip, 但是，实际上要获得用户的真实ip，不是只有这一个方法。

##### proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;

X-Forwarded-For 是一个 HTTP 扩展头部。HTTP/1.1（RFC 2616）协议并没有对它的定义，它最开始是由 Squid 这个缓存代理软件引入，用来表示 HTTP 请求端真实 IP。如今它已经成为事实上的标准，被各大 HTTP 代理、负载均衡等转发服务广泛使用，并被写入 [RFC 7239](http://tools.ietf.org/html/rfc7239)（Forwarded HTTP Extension）标准之中。

X-Forwarded-For 请求头格式非常简单，就这样：

```bash
X-Forwarded-For: client, proxy1, proxy2
```

可以看到，XFF 的内容由「英文逗号 + 空格」隔开的多个部分组成，最开始的是离服务端最远的设备 IP，然后是每一级代理设备的 IP。

 如果一个 HTTP 请求到达服务器之前，经过了三个代理 Proxy1、Proxy2、Proxy3，IP 分别为 IP1、IP2、IP3，用户真实 IP 为 IP0，那么按照 XFF 标准，服务端最终会收到以下信息：

  ```bash
X-Forwarded-For: IP0, IP1, IP2
  ```

Proxy3 直连服务器，它会给 XFF 追加 IP2，表示它是在帮 Proxy2 转发请求。列表中并没有 IP3，IP3 可以在服务端通过 Remote Address 字段获得。我们知道 HTTP 连接基于 TCP 连接，HTTP 协议中没有 IP 的概念，Remote Address 来自 TCP 连接，表示与服务端建立 TCP 连接的设备 IP，在这个例子里就是 IP3。

Remote Address 无法伪造，因为建立 TCP 连接需要三次握手，如果伪造了源 IP，无法建立 TCP 连接，更不会有后面的 HTTP 请求。不同语言获取 Remote Address 的方式不一样，例如 php 是 `$_SERVER["REMOTE_ADDR"]`，Node.js 是 `req.connection.remoteAddress`，但原理都一样。

对于 Web 应用来说，`X-Forwarded-For` 和 `X-Real-IP` 就是两个普通的请求头，自然就不做任何处理原样输出了。这说明，对于直连部署方式，除了从 TCP 连接中得到的 Remote Address 之外，请求头中携带的 IP 信息都不能信。  

##### proxy_set_header       Host $http_host;

- $host：请求中的主机头(HOST)字段，如果请求中的主机头不可用或者空，则为处理请求的server名称(处理请求的server的server_name指令的值)。值为小写，不包含端口!!!!

- 如果客户端发过来的请求的header中有’HOST’这个字段时，`$http_host`和`$host`都是原始的’HOST’字段比如请求的时候HOST的值是www.csdn.net 那么反代后还是www.csdn.net
  
  如果客户端发过来的请求的header中没有有’HOST’这个字段时， 建议使用$host，这表示请求中的server name。
  



# 4. websocket反向代理

+ nginx 首先确认版本必须是1.3以上。

+ map指令的作用： 该作用主要是根据客户端请求中$http_upgrade的值，来构造改变$connection_upgrade的值，即根据变量$http_upgrade的值创建新的变量$connection_upgrade， 创建的规则就是{}里面的东西。其中的规则没有做匹配，因此使用默认的，即 $connection_upgrade的值会一直是 upgrade。然后如果 $http_upgrade为空字符串的话， 那值会是 close。

```nginx
#必须添加的
map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
}

upstream websocket {
    ip_hash;
    #转发到服务器上相应的ws端口
    server localhost:3344;
    server localhost:8011;
}
server {
    listen 80;
    server_name a.liuvv.com;
    location / {
    
        #转发到http://websocket
        proxy_pass http://websocket;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
        #升级http1.1到 websocket协议  
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection  $connection_upgrade;
    }
}
```



# 5. 参考资料:

+ https://nginx.org/en/docs/
+ [Nginx配置location、if以及return、rewrite和 try_files 指令](https://www.xiebruce.top/710.html)
+ [Nginx 基本功能 - 将 Nginx 配置为 Web 服务器](https://blog.csdn.net/kikajack/article/details/79322194)
+ [Nginx 的 try_files 指令使用实例](https://www.hi-linux.com/posts/53878.html)
+ [HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)