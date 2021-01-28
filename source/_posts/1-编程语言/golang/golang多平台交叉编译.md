---
title: golang多平台交叉编译
tags: ["golang"]
abbrlink: 37af4aa1
categories:
  - 1-编程语言
  - golang
date: 2017-08-01 17:54:46
---


### cannot execute binary file exec format error

是因为mac和ubuntu的二进制格式不一致


### 问题

Go是一门编译型语言，所以在不同平台上，需要编译生成不同格式的二进制包。
由于Go 1.5对跨平台编译有了一些改进，包括统一了编译器、链接器等。
编译时候只需要指定两个参数：GOOS和GOARCH即可。


<!-- more -->


### 编译到 linux 64bit
```
$ GOOS=linux GOARCH=amd64 go build
```
### 或者可以使用 -o 选项指定生成二进制文件名字
```
$ GOOS=linux GOARCH=amd64 go build -o app.linux
```
### 编译到 linux 32bit
```
$ GOOS=linux GOARCH=386 go build
```
### 编译到 windows 64bit
```
$ GOOS=windows GOARCH=amd64 go build
```

### 编译到 windows 32bit
```
$ GOOS=windows GOARCH=386 go build
```

### 编译到 Mac OS X 64bit
```
$ GOOS=darwin GOARCH=amd64 go build
```