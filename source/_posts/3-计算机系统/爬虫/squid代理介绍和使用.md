---
title: squid代理介绍和使用
tags:
  - squid
  - 爬虫
categories:
  - 3-计算机系统
  - 爬虫
abbrlink: 94837db0
date: 2020-11-15 00:00:00
---

爬虫玩得好，牢饭吃到饱。

目前免费的代理池几乎不能用, 稳定的代理池收费又比较高。趁着双十一撸了几台便宜的云服务器, 直接拿来当代理服务器使用了。

squid服务程序是一款在Unix系统中最为流行的高性能代理服务软件，通常会被当作网站的前置缓存服务，用于替代用户向网站服务器请求页面数据并进行缓存，通俗来讲，squid服务程序会接收用户的请求，然后自动去下载制定数据(如网页)并存储在服务器内，当以后的用户再来请求相同数据时，则直接将刚刚存储在服务器本地的数据交给用户，较少用户的等待时间。

<!-- more -->

# 1. squid代理分类

### 1.1 正向代理

正向代理让用户可以通过Squid服务程序获取网站页面的数据，具体工作形式分为标准代理模式与透明代理模式。

+ 标准正向代理模式：

  用户访问网站，先到squid代理服务器，如果squid代理服务器有数据，那么squid就直接从缓存中放回给用户，如果squid缓存中没有，那么squid就代替用户去访问网站，把数据返回的给用户的同时留一份到缓存中，一遍下次用户（或者其他用户）访问的时候，直接从缓存中返回给用户

  

+ 透明正向代理模式：

  透明代理缓冲服务器和标准代理服务器的功能完全相同。相对于普通代理服务而言，客户端不需要做任何和代理服务器相关的设置，对用户而言，感觉不到代理服务器的存在，所以称之为透明代理。

  即把代理服务器部署在核心的上网出口，当用户上网浏览页面时，会交给代理服务器向外请求，如果结合iptables可以实现代理+网关+内容过滤+流量安全控制等完整的上网解决方案。

  

### 1.2 反向代理

反向代理是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从内部服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外表现为一个服务器。

squid做反向代理服务器，通常工作在一个服务器集群的前端，在用户看来，squid服务器就是他所要访问的服务器，而实际意义上，squid只是接受用户的请求，同时将用户请求转发给内部真正的WEB服务器，如果squid本身有用户要访问的内容，则squid直接将数据返回给用户，起到了缓存数据的作用，减少了后端服务器的压力。



### 1.3 区别

+ 概念

  正向代理：对于原始服务器而言，就是客户端的代言人

  反向代理：对于客户端而言，就像是原始服务器

+ 用途

  正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。正向代理还可以使用缓冲特性减少网络使用率。

  反向代理还可以为后端的多台服务器提供负载平衡，或为后端较慢的服务器提供缓冲服务。另外，反向代理还可以启用高级URL策略和管理技术，从而使处于不同web服务器系统的web页面同时存在于同一个URL空间下。

+ 安全性

  正向代理允许客户端通过它访问任意网站并且隐藏客户端自身，因此你必须采取安全措施以确保仅为经过授权的客户端提供服务。

  反向代理对外都是透明的，访问者并不知道自己访问的是一个代理。

  

# 2. 安装使用

```bash
apt update
apt install -y squid
```



### 2.1 修改配置

```bash
vi /etc/squid/squid.conf

# 代理服务器端口
http_port 3128


# 修改默认允许策略, 把 deny 换成 allow
http_access deny all
http_access allow all
```

修改完配置以后重启: `systemctl restart squid`



### 2.2 log 配置

squid的日志位于/var/log/squid/目录下。

```bash
tail -f /var/log/squid/access.log
```

如果想在log上加上日期: 

```ini
logformat combined %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh %{host}>h
access_log daemon:/var/log/squid/access.log combined
```



### 2.3 允许特定 ip

先增加一个规则, 然后允许

```ini
acl specialIP src x.x.x.x
http_access allow specialIP

# 例子
acl papa src 1.1.1.1
http_access allow papa
```



# 3. 注意事项

### 3.1 浏览器设置代理访问

TODO:

### 3.2 测试demo

如果无法访问, 请检查是否开启防火墙端口

```go
package main

import (
   "fmt"
   "io/ioutil"
   "log"
   "net/http"
   "net/url"
   "time"
)

func main() {
  proxy, err := url.Parse("http://代理ip:代理端口")
   if err != nil {
      log.Fatal(err)
   }

   httpClient := &http.Client{
      Timeout: time.Second * 10,
      Transport: &http.Transport{
         Proxy: http.ProxyURL(proxy),
      },
   }

   res, err := httpClient.Get("https://www.baidu.com")
   if err != nil {
      log.Println(err)
      return
   }
   defer res.Body.Close()
   if res.StatusCode != http.StatusOK {
      log.Println(err)
      return
   }
   c, _ := ioutil.ReadAll(res.Body)
   fmt.Println(string(c))
}
```



# 4. 参考资料

+ http://blog.haoji.me/linux-squid-proxy.html
+ https://www.cnblogs.com/bluestorm/p/9032086.html