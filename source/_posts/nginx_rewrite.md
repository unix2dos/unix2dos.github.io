---
title: nginx的rewrite规则
tags:
  - nginx
  - linux
abbrlink: 51e59d76
date: 2019-07-11 23:54:46
---



### 1. location指令

根据请求的URI来设置具体规则, URI是url中除去协议和域名及参数后, 剩下的部分.

比如请求的url为: http://www.liuvv.com/test/index.php?page=1, 则uri 为 `/test/index.php`

##### 1.1 location匹配uri的规则:

```
location [ = | ~ | ~* | ^~ ] uri { ... }
```

+ `=`  精确匹配, uri要完全一样
+ `^~`  这个是以某个开头, 不是正则匹配
+ `~`  区分大小写正则匹配
+ `~*` 不区分大小写正则匹配



<!-- more -->

##### 1.2 location匹配uri的优先级:

1. 首先先检查使用前缀字符定义的location，选择最长匹配的项并记录下来。

2. 如果找到了精确匹配的location，也就是使用了`=`修饰符的location，结束查找，使用它的配置。

3. 如果`^~`修饰符先匹配到最长的前缀字符串, 则不检查正则。

4. 然后按顺序查找使用正则定义的location，如果匹配则停止查找，使用它定义的配置。

5. 如果没有匹配的正则location，则使用前面记录的最长匹配前缀字符location。

   

> 基于以上的匹配过程，我们可以得到以下启示：

1. 使用正则定义的location在配置文件中出现的顺序很重要。因为找到第一个匹配的正则后，查找就停止了，后面定义的正则就是再匹配也没有机会了。
2. 使用精确匹配可以提高查找的速度。例如经常请求`/`的话，可以使用`=`来定义location。
3. 优先级 `=`  >  `^~`  >  正则



##### 1.3 location 测试

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



##### 1.4 location @name的用法

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



##### 1.5 URL尾部的`/`需不需要

关于URL尾部的`/`有三点也需要说明一下。第一点与location配置有关，其他两点无关。

1. location配置中的字符有没有`/`都没有影响。也就是说`/user/`和`/user`是一样的。
2. 如果URL结构是`https://domain.com/`的形式，尾部有没有`/`都不会造成重定向。因为浏览器在发起请求的时候，默认加上了`/`。虽然很多浏览器在地址栏里也不会显示`/`。这一点，可以访问[baidu](https://www.baidu.com/)验证一下。
3. 如果URL的结构是`https://domain.com/some-dir/`。尾部如果缺少`/`将导致重定向。因为根据约定，URL尾部的`/`表示目录，没有`/`表示文件。所以访问`/some-dir/`时，服务器会自动去该目录下找对应的默认文件。如果访问`/some-dir`的话，服务器会先去找`some-dir`文件，找不到的话会将`some-dir`当成目录，重定向到`/some-dir/`，去该目录下找默认文件。



### 2. rewirte规则


##### 2.1 return指令

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



##### 2.2 rewrite指令

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



##### 2.3 try_files

try_files指令写在server和location里面.

```nginx
ry_files file ... uri 或 try_files file ... = code
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



##### 2.4 if指令

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



##### 2.5 alias

nginx指定文件路径有两种方式root和alias.

+ root

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

+ alias

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



### 参考资料:

+ https://nginx.org/en/docs/

+ https://www.xiebruce.top/710.html

+ https://blog.csdn.net/kikajack/article/details/79322194

+ https://www.hi-linux.com/posts/53878.html