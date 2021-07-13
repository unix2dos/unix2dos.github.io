---
title: golang启动https
tags:
  - http
  - linux
  - golang
categories:
  - 3-计算机系统
  - https
abbrlink: d5ecb4c4
date: 2019-07-18 19:09:03
---



# 1. golang 实现HTTPS Web Server

+ 生成私钥和证书

```
openssl genrsa -out server.key 2048 //生成私钥
openssl req -new -x509 -key server.key -out server.pem -days 3650 //生成证书
```
<!-- more -->

+ server.go

```
package main
	
import (
	"fmt"
	"net/http"
)
	
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hi, This is an example of https service in golang!")
}
	
func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServeTLS(":8081", "server.pem", "server.key", nil)
}
	
```

通过浏览器访问：https://localhost:8081 会出现您的连接不是私密连接, 因为我们使用的是自签发的数字证书
	

+ 客户端访问

```
package main
	
import (
	"crypto/tls"
	"fmt"
	"io/ioutil"
	"net/http"
)
	
func main() {
	//通过设置tls.Config的InsecureSkipVerify为true，client将不再对服务端的证书进行校验。
	ts := &http.Transport{TLSClientConfig: &tls.Config{InsecureSkipVerify: true}} 
	client := &http.Client{Transport: ts}
	
	resp, err := client.Get("https://localhost:8081")
	if err != nil {
		fmt.Println("error:", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```



# 2. 对服务端数字证书进行验证

接下来我们来验证一下客户端对服务端数字证书进行验证:

- 首先我们来建立我们自己的CA，需要生成一个CA私钥和一个CA的数字证书:
```
openssl genrsa -out ca.key 2048
	
openssl req -x509 -new -nodes -key ca.key -subj "/CN=tonybai.com" -days 5000 -out ca.crt
```

- 接下来，生成server端的私钥，生成数字证书请求，并用我们的ca私钥签发server的数字证书：

```
openssl genrsa -out server.key 2048
	
openssl req -new -key server.key -subj "/CN=localhost" -out server.csr
	
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000
```

- 现在我们的工作目录下有如下一些私钥和证书文件：

```
CA:
私钥文件 ca.key
数字证书 ca.crt

Server:
私钥文件 server.key
数字证书 server.crt
```

+ 客户端验证服务端数字证书

```
package main
	
import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io/ioutil"
	"net/http"
)
	
func main() {
	
	pool := x509.NewCertPool()
	caCrt, err := ioutil.ReadFile("ca.crt")
	if err != nil {
		fmt.Println("ReadFile err:", err)
		return
	}
	pool.AppendCertsFromPEM(caCrt)
	
	ts := &http.Transport{
		TLSClientConfig: &tls.Config{
			RootCAs:            pool,
			InsecureSkipVerify: false,
		},
	}
	client := &http.Client{Transport: ts}
	
	resp, err := client.Get("https://localhost:8081")
	if err != nil {
		fmt.Println("error:", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```



# 3. 对客户端的证书进行校验(双向证书校验）

+ 要对客户端数字证书进行校验，首先客户端需要先有自己的证书。

```
openssl genrsa -out client.key 2048
	
openssl req -new -key client.key -subj "/CN=tonybai_cn" -out client.csr
	
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 5000
```

+ 首先server端需要要求校验client端的数字证书，并且加载用于校验数字证书的ca.crt，因此我们需要对server进行更加灵活的控制：

```
package main
	
import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io/ioutil"
	"net/http"
)
	
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hi, This is an example of https service in golang!")
}
	
func main() {
	pool := x509.NewCertPool()
	caCrt, err := ioutil.ReadFile("ca.crt")
	if err != nil {
		fmt.Println("ReadFile err:", err)
		return
	}
	pool.AppendCertsFromPEM(caCrt)
	
	s := &http.Server{
		Addr:    ":8081",
		Handler: http.HandlerFunc(handler),
		TLSConfig: &tls.Config{
			ClientCAs:  pool,
			ClientAuth: tls.RequireAndVerifyClientCert, //强制校验client端证书
		},
	}
	
	s.ListenAndServeTLS("server.crt", "server.key")
}
```

+ client端变化也很大，需要加载client.key和client.crt用于server端连接时的证书校验：

```
package main
	
import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io/ioutil"
	"net/http"
)
	
func main() {
	
	pool := x509.NewCertPool()
	caCrt, err := ioutil.ReadFile("ca.crt")
	if err != nil {
		fmt.Println("ReadFile err:", err)
		return
	}
	pool.AppendCertsFromPEM(caCrt)
	
	cliCrt, err := tls.LoadX509KeyPair("client.crt", "client.key")
	if err != nil {
		fmt.Println("Loadx509keypair err:", err)
		return
	}
	
	ts := &http.Transport{
		TLSClientConfig: &tls.Config{
			RootCAs:            pool, //client端是 RootCAs
			Certificates:       []tls.Certificate{cliCrt},
			InsecureSkipVerify: false,
		},
	}
	client := &http.Client{Transport: ts}
	
	resp, err := client.Get("https://localhost:8081")
	if err != nil {
		fmt.Println("error:", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```