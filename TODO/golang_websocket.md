---
title: "golang的websocket的使用"
date: 2019-08-20 17:54:46
tags:
- golang
- websocket
---



这里先简单介绍一下 websocket，确实只是简单介绍一下。

## 1. 应用场景

有些场景下，比如交易 K 线，我们需要前端对后端进行轮询来不断获取或者更新资源状态。轮询的问题毫无以为是一种笨重的方式，因为每一次 http 请求除了本身的资源信息传输外还有三次握手以及四次挥手。替代轮询的一种方案是复用一个 http 连接，更准确的复用同一个 tcp 连接。这种方式可以是 http 长连接，也可以是 websocket。

## 2. websocket 和 http 长连接的区别

首先 websocket 和 http 是完全不同的两种协议，虽然底层都是 tcp/ip。http 长连接也是属于 http 协议。http 协议和 websocket 的最大区别就是 http 是基于 request/response 模式，而 websocket 的 client 和 server 端却可以随意发起 data push，比如服务端向 app 端的消息下发就比较适合使用 websocket（这种场景下使用 http 长连接也是可以，client 端定时向 server 端发送消息，比如 heatbeat，然后 server 端要 push 的消息以 response 的形式返回给 client）。







3. http长短连接

不存在长短链接, tcp才存在

HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接。

HTTP会话永远都是：请求响应结束，这里长连接指一次TCP连接可以传递多次的HTTP报文信息。











WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。
WebSocket通信协议于2011年被IETF定为标准RFC 6455，并被RFC7936所补充规范。

在golang语言中，目前有两种比较常用的实现方式：一个是golang自带的库，另一个是[gorilla](github.com/gorilla/websocket)，功能强大。

<!-- more -->

### 1. server端

server端是一个HTTP 服务器，监听8080端口。

当接收到连接请求后，将连接使用的http协议升级为websocket协议。后续通信过程中，使用websocket进行通信。

对每个连接，server端等待读取数据，读到数据后，打印数据，然后，将数据又发送给client.



```go
// +build ignore

package main

import (
	"flag"
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

var addr = flag.String("addr", "localhost:8080", "http service address")

var upgrader = websocket.Upgrader{} // use default options

func echo(w http.ResponseWriter, r *http.Request) {
	c, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Print("upgrade:", err)
		return
	}
	defer c.Close()
	for {
		mt, message, err := c.ReadMessage()
		if err != nil {
			log.Println("read:", err)
			break
		}
		log.Printf("server recv: %s", message)
		err = c.WriteMessage(mt, message)
		if err != nil {
			log.Println("write:", err)
			break
		}
	}
}

func main() {
	flag.Parse()
	http.HandleFunc("/echo", echo)
	log.Fatal(http.ListenAndServe(*addr, nil))
}
```



### 2. client端

client启动后，首先连接server。

连接建立后，主routine每一秒钟向server发送消息(当前时间)。另一个routine从server接收数据,并打印。

当client退出时，会向server发送关闭消息。接着，等待退出。

```go
// +build ignore

package main

import (
	"flag"
	"log"
	"net/url"
	"os"
	"os/signal"
	"time"

	"github.com/gorilla/websocket"
)

var addr = flag.String("addr", "localhost:8080", "http service address")

func main() {
	flag.Parse()
	log.SetFlags(0)

	interrupt := make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)

	u := url.URL{Scheme: "ws", Host: *addr, Path: "/echo"}
	log.Printf("connecting to %s", u.String())

	c, _, err := websocket.DefaultDialer.Dial(u.String(), nil)
	if err != nil {
		log.Fatal("dial:", err)
	}
	defer c.Close()

	done := make(chan struct{})

	go func() {
		defer close(done)
		for {
			_, message, err := c.ReadMessage()
			if err != nil {
				log.Println("read:", err)
				return
			}
			log.Printf("client recv: %s", message)
		}
	}()

	ticker := time.NewTicker(time.Second)
	defer ticker.Stop()

	for {
		select {
		case <-done:
			return
		case t := <-ticker.C:
			err := c.WriteMessage(websocket.TextMessage, []byte(t.String()))
			if err != nil {
				log.Println("write:", err)
				return
			}
		case <-interrupt:
			log.Println("interrupt")

			// Cleanly close the connection by sending a close message and then
			// waiting (with timeout) for the server to close the connection.
			err := c.WriteMessage(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""))
			if err != nil {
				log.Println("write close:", err)
				return
			}
			select {
			case <-done:
			case <-time.After(time.Second):
			}
			return
		}
	}
}
```



![1](golang_websocket/1.png)



### 3. 总结

##### 服务器:

+ var upgrader = websocket.Upgrader{}
+ c, err := upgrader.Upgrade(w, r, nil)
+ for循环里 c.ReadMessage()  和 c.WriteMessage(mt, message)



##### 客户端:

+ c, _, err := websocket.DefaultDialer.Dial(u.String(), nil)
+ for循环里 c.ReadMessage()  和 c.WriteMessage(mt, message)





### 4. 参考资料

+ https://zhuanlan.zhihu.com/p/35167916
+ https://www.jianshu.com/p/3fc3646fad80