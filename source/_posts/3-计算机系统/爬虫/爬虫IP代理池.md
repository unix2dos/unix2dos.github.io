---
title: 爬虫IP代理池
tags:
  - 爬虫
categories:
  - 3-计算机系统
  - 爬虫
date: 2020-10-23 00:00:00
---



对于爬虫来说，由于爬虫爬取速度过快，在爬取过程中可能遇到同一个IP访问过于频繁的问题，此时网站就会让我们输入验证码登录或者直接封锁IP，这样会给爬取带来极大的不便。

使用代理隐藏真实的IP，让服务器误以为是代理服务器在请求自己。这样在爬取过程中通过不断更换代理，就不会被封锁，可以达到很好的爬取效果。

<!-- more -->



# 1. 免费代理

免费代理大多数情况下都是不好用的，所以比较靠谱的方法是购买付费代理。

### 1.1 开源项目 proxy_pool

##### 1.1.1 安装和启动

``` bash
docker pull jhao104/proxy_pool

#依赖redis, 所以要先安装 redis
docker run -d --name redis -p 6379:6379 redis:latest  --requirepass "123456"
docker run --env DB_CONN=redis://:123456@172.17.0.1:6379/1 -p 5010:5010 --name proxy_pool jhao104/proxy_pool:latest
```



#### 1.1.2 使用

docker 启动后, 通过访问本地的接口, 就可以得到 ip

```
http://127.0.0.1:5010/get/
```

或者用作者给出的测试地址: http://118.24.52.95/get/




# 2. 付费代理

### 2.1 芝麻代理

+ http://h.zhimaruanjian.com/getapi/

+ 生成 API 链接

+ 调用 API 获取 IP 和端口(如果提示加入白名单, 用文档里的接口即可)

  

# 3. 代码

### 3.1 golang 

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

+ https://cuiqingcai.com/5491.html
+ https://github.com/jhao104/proxy_pool
+ http://www.zhimaruanjian.com/