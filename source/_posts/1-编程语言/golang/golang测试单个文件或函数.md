---
title: golang测试单个文件或函数
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: 401250d7
date: 2020-05-13 00:00:00
---

### 1. 测试单个文件或函数

测试一个文件

```bash
go test -v hello_test.go
```



测试一个函数

```bash
go test -v  -test.run="TestA"  
```

<!-- more -->

注意在测试单个文件时, 会出现未定义的情况, 这是因为定义在其他文件里, 需要加上定义的文件.

```bash
go test -v hello.go hello_test.go
```



而测试单个函数不存在这个问题, 可以在一个文件内用相同的前缀命名测试函数, 然后用正则表达式去测试.

如:

```bash
go test -v  -test.run="TestA*"  
```



### 2. 测试覆盖率

```bash
go test -v -coverprofile=a.out -test.run="TestA*" # 把测试结果保存在 a.out

go tool cover -html=./a.out  # 通过浏览器打开, 可以看到覆盖经过的函数
```



### 3. 总结

不写单元测试的代码都是耍流氓.