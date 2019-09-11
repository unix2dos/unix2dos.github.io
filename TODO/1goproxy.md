
git clone https://github.com/gomods/athens 

cd athens 

make build-ver VERSION="0.5.0"

./athens -version







服务器开启

export ATHENS_STORAGE_TYPE=disk

export ATHENS_DISK_STORAGE_ROOT=~/athens-storage

./athens





客户端

export GO111MODULE=on export GOPROXY=[http://127.0.0.1:3000](http://127.0.0.1:3000/)





git clone https://github.com/athens-artifacts/walkthrough.git 

cd walkthrough

go run .













Now, if you view the contents of the athens_storage directory, you will see that you now have additional files representing the samplelib module.





ls -lr $ATHENS_STORAGE/github.com/athens-artifacts/samplelib/v1.0.0/





sudo rm -fr "$(go env GOPATH)/pkg/mod" 

我们再来一次 go run .







curl 127.0.0.1:3000/github.com/athens-artifacts/samplelib/@v/list







https://tonybai.com/2018/11/26/hello-go-module-proxy/

https://juejin.im/post/5c8f9f8ef265da612c3a34b9

https://docs.gomods.io/

https://research.swtch.com/vgo-module