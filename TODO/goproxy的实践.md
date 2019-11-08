







### 1.  goproxy

https://github.com/goproxyio/goproxy



##### 1.1 安装go

```bash
wget https://dl.google.com/go/go1.13.4.linux-amd64.tar.gz #下载go
tar -C /usr/local -xzf go1.13.4.linux-amd64.tar.gz # 解压到/usr/local
export PATH=$PATH:/usr/local/go/bin # 设置环境变量
go version # go version go1.13.4 linux/amd64
```



##### 1.2 安装goproxy

```bash
git clone https://github.com/goproxyio/goproxy.git
cd goproxy/
make
```



##### 1.3 代理

```bash
# go.1.12.x
export GO111MODULE=on
export GOPROXY=https://goproxy.io

# go1.13.x
go env -w GOPROXY=https://goproxy.io,direct
go env -w GOPRIVATE=*.corp.example.com


# 执行
./bin/goproxy -cacheDir=/var/opt/goproxy -listen 0.0.0.0:8082 
./bin/goproxy -listen=0.0.0.0:8082 -cacheDir=/tmp/test

# 操作
GOPROXY=http://localhost:8082 go get -v github.com/spf13/cobra
```







```
./bin/goproxy -listen=0.0.0.0:80 -cacheDir=/tmp/test -proxy https://goproxy.io -exclude "*.corp.example.com,rsc.io/private"



./bin/goproxy -cacheDir /var/opt/goproxy -listen :8082 -proxy https://goproxy.cn -exclude liuvv.com


server {
    server_name goproxy.fhyx.tech;
    listen 80 ;
    listen 443 ssl http2 ;
    access_log /var/log/nginx/goproxy_access_log;
    error_log /var/log/nginx/goproxy_error_log notice;

    location  / {
        proxy_set_header       X-Real-IP $remote_addr;
        proxy_set_header       X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header       Host $http_host;
        proxy_connect_timeout 300s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;

        proxy_pass             http://127.0.0.1:8082;
    }
}
```





### 2.  Athens

https://github.com/gomods/athens



```
git clone https://github.com/gomods/athens 

cd athens 

make build-ver VERSION="0.5.0"

./athens -version

```







服务器开启

```
export ATHENS_STORAGE_TYPE=disk

export ATHENS_DISK_STORAGE_ROOT=~/athens-storage

./athens

```









客户端

```
export GO111MODULE=on export GOPROXY=http://127.0.0.1:3000





git clone https://github.com/athens-artifacts/walkthrough.git 

cd walkthrough

go run .



```









Now, if you view the contents of the athens_storage directory, you will see that you now have additional files representing the samplelib module.





ls -lr $ATHENS_STORAGE/github.com/athens-artifacts/samplelib/v1.0.0/





sudo rm -fr "$(go env GOPATH)/pkg/mod" 

我们再来一次 go run .







curl 127.0.0.1:3000/github.com/athens-artifacts/samplelib/@v/list





### 3. 参考资料

+ [Hello，Go module proxy](https://tonybai.com/2018/11/26/hello-go-module-proxy/)

+ [Go Module Proxy](https://juejin.im/post/5c8f9f8ef265da612c3a34b9)

+ https://docs.gomods.io/