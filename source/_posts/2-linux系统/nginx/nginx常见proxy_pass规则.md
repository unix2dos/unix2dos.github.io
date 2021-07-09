---
title: nginx常见proxy_pass规则
tags:
  - nginx
categories:
  - 2-linux系统
  - nginx
abbrlink: '55856485'
date: 2021-02-28 00:00:00
---

记录一些常用的nginx转发规则记录

<!-- more -->

# 1. 有无 / 结尾

在location中匹配的url最后有无/结尾，指的是模糊匹配与精确匹配的问题
在proxy_pass中代理的url最后有无/结尾，指的是在proxy_pass 指定的url后要不要加上location匹配的url的问题

### 1.1 localtion 加不加 /

+ location /abc/def  

  可以匹配/abc/defghi的请求，也可以匹配/abc/def/ghi ......

+ location /abc/def/  

  不能匹配/abc/defghi的请求，只能精确匹配 /abc/def/ghi这样的请求

### 1.2 proxy_pass 加不加/

```nginx
location /star/ {
	proxy_pass http://ent.163.com;
}

#如果是location /star/，http://abc.163.com/star/1.html ==> http://ent.163.com/star/1.html。
#如果是location /blog/，http://abc.163.com/blog/1.html ==> http://ent.163.com/blog/1.html。
```

location是什么，nginx 就把location 加在proxy_pass 的 server 后面 。



```nginx
location /star/ {
	proxy_pass http://ent.163.com/;
}

#如果是location /star/, 	http://abc.163.com/star/1.html ==> http://ent.163.com/1.html。
#如果是location /blog/,  http://abc.163.com/blog/1.html ==> http://ent.163.com/1.html。
```


改变location，并不能改变返回的内容，返回的内容始终是http://ent.163.com/ 。




# 2. 匹配并转发参数

需求是 `/api/camps/v1/teacher`   -> 其他服务器  `/api/v1/teacher `, 并且转发参数

```nginx
location ~ ^/api/camps/v1/(.*)$ {
            include /etc/nginx/proxy_params;
            proxy_pass http://127.0.0.1:9082/api/v1/$1$is_args$args;
  }
```



# 3. 代理相对路径

### 3.1 完全转发

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location / {
                proxy_pass https://www.liuvv.com/;
        }

}
```

http://159.75.75.191/  此时是完全代理的

### 3.2 二级转发

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location /liuvv/ {
                proxy_pass https://www.liuvv.com/;
        }
}
```

http://159.75.75.191/liuvv/  此时一些子url 是404, 因为直接打到了 `/` 路径

##### 3.2.1 方案一

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location /liuvv/ {
                proxy_pass https://www.liuvv.com/;
        }
	       location / {
               proxy_pass https://www.liuvv.com/;
       }
}
```

##### 3.2.1 方案二

```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        location ^~ /liuvv/ {
        	proxy_pass https://www.liuvv.com/;
        }
        location ~ ^/([A-Za-z0-9]+) {
        	proxy_pass https://www.liuvv.com;
        }
        location / {
    			return 403;
        }
}
```



# 4. 参考资料

+ https://stackoverflow.com/a/8130872/7062454
+ https://serverfault.com/a/932636
+ https://www.liuvv.com/p/51e59d76.html
