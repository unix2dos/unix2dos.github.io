---
title: go-module的使用
tags:
  - golang
abbrlink: 6c6632b7
date: 2019-06-03 15:56:46
---



golang从诞生之初就一直有个被诟病的问题：缺少一个行之有效的“官方”包依赖管理工具。之前golang包管理工具有数十个， 说实话都不是让人非常满意。

go 1.11 有了对模块的实验性支持，大部分的子命令都知道如何处理一个模块，比如 run build install get list mod 子命令。go 1.12 会删除对 GOPATH 的支持，go get 命令也会变成只能获取模块，不能像现在这样直接获取一个裸包。





可以用环境变量 GO111MODULE 开启或关闭模块支持，它有三个可选值：off、on、auto，默认值是 auto。

- GO111MODULE=off 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。
- GO111MODULE=on 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 go.mod 下载依赖。
- GO111MODULE=auto 在 $GOPATH/src 外面且根目录有 go.mod 文件时，开启模块支持。

在使用模块的时候，GOPATH 是无意义的，不过它还是会把下载的依赖储存在 $GOPATH/pkg/mod 中，也会把 go install 的结果放在 $GOPATH/bin 中。

<!-- more -->

### 1. go mod 使用教程

https://github.com/golang/go/wiki/Modules # 官方wiki, 基本所有的问题都能在这里找到



##### 1.1 在使用前先确保golang升级到1.11:

https://golang.org/dl/ #下载golang



### 2. 使用go mod 实践



##### 2.1. 快速开始：

1. 把之前的工程拷贝到$GOPATH/src之外

2. 在工程目录下执行：go mod init {module name}，该命令会创建一个go.mod文件

3. 然后在该目录下执行 go build，就可以了，go.mod中记录了依赖包及其版本号。

   

##### 2.2. 在 go build 中 遇到了以下几个问题, 记录如下:



2.2.1 golang.org的包竟然下不下来(你懂的)

可以使用在go.mod里添加replace选项

```go
replace (golang.org/x/text => github.com/golang/text v0.3.0 )
```

 也可以用代理的方式, 更加方便

```shell
export GOPROXY=https://athens.azurefd.net
```



2.2.2 发现找不到包(包层级多的)

cannot load github.com/aliyun/alibaba-cloud-sdk-go/sdk: cannot find module providing package github.com/aliyun/alibaba-cloud-sdk-go/sdk

```shell
go get github.com/aliyun/alibaba-cloud-sdk-go@master  #带@master
```



2.2.3 在替换的时候还发现这个错误

cannot call non-function xurls.Strict (type *regexp.Regexp)

```shell
mvdan.cc/xurls  #之前用的是这个版本,换成下面的就可以了
mvdan.cc/xurls/v2
```



2.2.4  只有直接使用的依赖会被记录在`go.mod`文件中, 贴出go.mod的内容如下:

```go
module github.com/unix2dos/goods-notify



go 1.12



require (

	github.com/PuerkitoBio/goquery v1.5.0

	github.com/aliyun/alibaba-cloud-sdk-go v0.0.0-20190528035818-94084c920892

	github.com/gorilla/websocket v1.4.0 // indirect

	github.com/joho/godotenv v1.3.0

	github.com/pkg/errors v0.8.1 // indirect

	github.com/stretchr/testify v1.3.0

	github.com/unix2dos/bearychat-go v0.0.0-20190222142113-d09d4a5e73e5

	github.com/valyala/fasthttp v1.3.0

	github.com/yunpian/yunpian-go-sdk v0.0.0-20171206021512-2193bf8a7459

	golang.org/x/text v0.3.2

	mvdan.cc/xurls/v2 v2.0.0

)
```



`indirect` 注释标记了依赖不是被当前模块直接使用的，只是在其他依赖项中被间接引用。



2.2.5  go.sum文件介绍

同时，`go.mod`和`go`命令维护了一个名叫`go.sum`的文件包含了指定模块版本的期望的[加密hash](https://golang.org/cmd/go/#hdr-Module_downloading_and_verification)：

`go`命令使用`go.sum`文件保证之后的模块下载会下载到跟第一次下载相同的文件内容，保证你的项目依赖不会发生预期外的恶意修改、意外问题和其他问题。`go.mod` 和 `go.sum`都需要放进版本管理中。



### 3. go mod 相关操作

`go list -m all`  可以查看当前的依赖和版本(当前模块，或者叫做主模块，通常是第一行，接下来是根据依赖路径排序的依赖)。



`go mod edit -fmt` 格式化 `go.mod` 文件。



 `go mod tidy` 从 `go.mod` 删除不需要的依赖、新增需要的依赖，这个操作不会改变依赖版本。



##### 3.1 go get 命令

获取依赖的特定版本，用来升级和降级依赖。可以自动修改 `go.mod` 文件，而且依赖的依赖版本号也可能会变。在 `go.mod` 中使用 `exclude` 排除的包，不能 `go get` 下来。



与以前不同的是，新版 `go get` 可以在末尾加 `@` 符号，用来指定版本。

它要求仓库必须用 `v1.2.0` 格式打 tag，像 `v1.2` 少个零都不行的，必须是[语义化](https://semver.org/lang/zh-CN/)的、带 `v` 前缀的版本号。



```
go get github.com/gorilla/mux           # 匹配最新的一个 tag
go get github.com/gorilla/mux@latest    # 和上面一样
go get github.com/gorilla/mux@v1.6.2    # 匹配 v1.6.2
go get github.com/gorilla/mux@e3702bed2 # 匹配 v1.6.2
go get github.com/gorilla/mux@c856192   # 匹配 c85619274f5d
go get github.com/gorilla/mux@master    # 匹配 master 分支
```



`latest` 匹配最新的 tag。

`v1.2.6` 完整版本的写法。

`v1`、`v1.2` 匹配带这个前缀的最新版本，如果最新版是 `1.2.7`，它们会匹配 `1.2.7`。

`c856192` 版本 hash 前缀、分支名、无语义化的标签，在 `go.mod` 里都会会使用约定写法 `v0.0.0-20180517173623-c85619274f5d`，也被称作伪版本。

`go get` 可以模糊匹配版本号，但 `go.mod` 文件只体现完整的版本号，即 `v1.2.0`、`v0.0.0-20180517173623-c85619274f5d`，只不过不需要手写这么长的版本号，用 `go get` 或上文的 `go mod edit -require` 模糊匹配即可，它会把匹配到的完整版本号写进 `go.mod`  文件。





参考资料:

https://github.com/golang/go/wiki/Modules

https://www.jianshu.com/p/c5733da150c6

https://www.4async.com/2019/03/using-go-modules/