---
title: session的介绍和使用
tags:
  - session
  - golang
categories:
  - 3-计算机系统
  - Web
abbrlink: 312a7a36
date: 2019-09-12 22:59:16
---



### 0. 前言

除了使用Cookie，Web应用程序中还经常使用Session来记录客户端状态。Session是服务器端使用的一种记录客户端状态的机制，使用上比Cookie简单一些，相应的也增加了服务器的存储压力。



Session在用户第一次访问服务器的时候自动创建。客户端只保存sessionid到cookie中，而不会保存session，session销毁只能通过invalidate或超时(默认30分钟)，关掉浏览器并不会关闭session。

<!-- more -->

### 1. session 介绍

##### 1.1 Session与Cookie的区别

cookie与session最大的区别就是一个是将数据存放在客户端，一个是将数据存放在服务端。cookie是将信息都存放在客户端的浏览器内存或磁盘中，所以不是很安全，别人可以分析存放在本地的cookie数据来进行用户信息的盗窃或进行cookie欺骗。

所以在安全性上session要好一些，session通信的一般实现形式是通过cookie来实现，与cookie不同的是，session只会保存一个sessionID在客户端，不会像cookie那样将具体的数据保存在客户端，session具体的数据只会保存在服务端上

在Servlet中session数据是被封装在一个对象里，而这个对象会被保存在对象池中，客户端发生请求时会带上它的sessionID，服务端就会根据这个sessionID，来从对象池中获得相应的session对象，从对象中获得session的具体数据，服务端通过这个session数据来保持或改变与客户端会话的状态。



##### 1.2 Session机制

以上也介绍了Session有两个主要的东西，一个是SessionID，一个是存放在服务端对象池中的Session对象。

客户端访问服务端的时候，会先判断这个客户端的请求数据中是否包含有SessionID，如果没有的话，就会认为这个客户端是第一次进行访问。因为是第一次访问，所以服务端会给客户端在对象池中创建一个Session对象（假设这个会话是需要维持的），并生成出这个对象的SessionID，接着会通过cookie将SessionID响应给客户端，同时会把Session对象放回对象池里。客户端接收响应数据后会将SessionID存放在本地，下一次再访问服务端的时候就会把SessionID给带上，服务端就能够通过SessionID获得相应的Session对象，Session就是以这样的一个机制维持会话状态的。



##### 1.3 session 存储问题

session如何存储才是高效，是存在内存、文件还是数据库了？文件和数据库的存储方式都是将session的数据固化到硬盘上，操作硬盘的方式就是IO，IO操作的效率是远远低于操作内存的数据，因此文件和数据库存储方式是不可取的，所以将session数据存储到内存是最佳的选择。

因此最好的解决方案就是使用分布式缓存技术，例如：memcached和redis，将session信息的存储独立出来也是解决session同步问题的方法。



##### 1.4 session劫持防范

其中一个解决方案就是sessionID的值只允许cookie设置，而不是通过URL重置方式设置，同时设置cookie的httponly为true,这个属性是设置是否可通过客户端脚本访问这个设置的cookie，第一这个可以防止这个cookie被XSS读取从而引起session劫持，第二cookie设置不会像URL重置方式那么容易获取sessionID。

第二步就是在每个请求里面加上token，实现类似前面章节里面讲的防止form重复递交类似的功能，我们在每个请求里面加上一个隐藏的token，然后每次验证这个token，从而保证用户的请求都是唯一性。

还有一个解决方案就是，我们给session额外设置一个创建时间的值，一旦过了一定的时间，我们销毁这个sessionID，重新生成新的session，这样可以一定程度上防止session劫持的问题。



### 2. golang 使用 session

+ github.com/gorilla/sessions

```go
// sessions.go
package main

import (
    "fmt"
    "net/http"

    "github.com/gorilla/sessions"
)

var (
    // key must be 16, 24 or 32 bytes long (AES-128, AES-192 or AES-256)
    key = []byte("super-secret-key")
    store = sessions.NewCookieStore(key)
)

func secret(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "cookie-name")

    // Check if user is authenticated
    if auth, ok := session.Values["authenticated"].(bool); !ok || !auth {
        http.Error(w, "Forbidden", http.StatusForbidden)
        return
    }

    // Print secret message
    fmt.Fprintln(w, "The cake is a lie!")
}

func login(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "cookie-name")

    // Authentication goes here
    // ...

    // Set user as authenticated
    session.Values["authenticated"] = true
    session.Save(r, w)
}

func logout(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "cookie-name")

    // Revoke users authentication
    session.Values["authenticated"] = false
    session.Save(r, w)
}

func main() {
    http.HandleFunc("/secret", secret)
    http.HandleFunc("/login", login)
    http.HandleFunc("/logout", logout)

    http.ListenAndServe(":8080", nil)
}
```



```bash
$ go run sessions.go

$ curl -s http://localhost:8080/secret
Forbidden

$ curl -s -I http://localhost:8080/login
Set-Cookie: cookie-name=MTQ4NzE5Mz...

$ curl -s --cookie "cookie-name=MTQ4NzE5Mz..." http://localhost:8080/secret
The cake is a lie!
```



+ https://github.com/gin-contrib/sessions

```go
package main

import (
	"github.com/gin-contrib/sessions"
	"github.com/gin-contrib/sessions/cookie"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	store := cookie.NewStore([]byte("secret"))
	r.Use(sessions.Sessions("mysession", store))

	r.GET("/incr", func(c *gin.Context) {
		session := sessions.Default(c)
		var count int
		v := session.Get("count")
		if v == nil {
			count = 0
		} else {
			count = v.(int)
			count++
		}
		session.Set("count", count)
		session.Save()
		c.JSON(200, gin.H{"count": count})
	})
	r.Run(":8000")
}
```



### 3. 参考资料

+ https://www.iteye.com/blog/justsee-1570652
+ https://gowebexamples.com/sessions/
+ https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/06.0.md



