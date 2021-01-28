---
title: golang编写测试用例
tags: ["golang"]
categories:
  - 1-编程语言
  - golang
abbrlink: 269bf134
date: 2019-08-24 07:37:46
---



### 1. Learn Go with tests

当学习一门语言时, 最有效的办法不是每一章的去阅读概念, 而是通过例子探索学习.

如果没有学习过 Go 语言的, 强烈建议通过编写测试学习 Go 语言, 不仅为测试驱动开发打下基础, 还是可以使用 Go 语言编写健壮的、经过良好测试的系统.

强烈推荐: https://github.com/quii/learn-go-with-tests

<!-- more -->



### 2. Golang Test

Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试，`testing`框架和其他语言中的测试框架类似，你可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例。测试用例有四种形式： 

+ TestXxxx(t *testing.T) // 单元测试
+ TestBenchmarkXxxx(b* testing.B) // 压力测试
+ Example_Xxx() // 测试控制台输出的例子 
+ TestMain(m *testing.M) // 测试Main函数

当然我们也可以使用第三方的测试框架, 更加高效的测试我们的代码:

https://github.com/stretchr/testify



###  3. 单元测试


+ 需要创建一个名称以 _test.go 结尾的文件，该文件包含 `TestXxx` 函数
+ `func TestXxx(*testing.T)`   // Xxx 可以是任何字母数字字符串，但是第一个字母不能是小些字母。
+ 单元测试中，传递给测试函数的参数是 `*testing.T` 类型。



##### 3.1 单元测试方法

+ 当我们遇到一个断言错误的时候，标识这个测试失败，会使用到：

  ```
  Fail: 测试失败，测试继续，也就是之后的代码依然会执行
  FailNow: 测试失败，测试中断
  ```

+ 当我们只希望打印信息，会用到:

  ```
  Log: 输出信息
  Logf: 输出格式化的信息
  ```

+ 当我们断言失败的时候，不希望标识测试失败，会用到：

  ```
  Skip: 相当于 Log + SkipNow
  Skipf: 相当于 Logf + SkipNow
  SkipNow: 跳过测试，测试中断
  ```

+ 当我们断言失败的时候，希望标识测试失败，但是测试继续，会用到：

  ```
  Error: 相当于 Log + Fail
  Errorf: 相当于 Logf + Fail
  ```

+ 当我们断言失败的时候，希望标识测试失败，但中断测试，会用到

  ```
  Fatal: 相当于 Log + FailNow
  Fatalf: 相当于 Logf + FailNow
  ```

  

### 4.  压力测试

+ func BenchmarkXxx(*testing.B)  //函数形式
+ 通过 "go test" 命令，加上 `-bench` flag 来执行

```go
func BenchmarkIsPalindrome(b *testing.B) {
	for i := 0; i < b.N; i++ {
		IsPalindrome("A man, a plan, a canal: Panama")
	}
}

$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 1000000                1035 ns/op
ok      gopl.io/ch11/word2      2.179s
```

结果中基准测试名的数字后缀部分，这里是8，表示运行时对应的GOMAXPROCS的值。

报告显示每次调用IsPalindrome函数花费1.035微秒，是执行1,000,000次的平均时间。

因为基准测试驱动器开始时并不知道每个基准测试函数运行所花的时间，它会尝试在真正运行基准测试前先尝试用较小的N运行测试来估算基准测试函数所需要的时间，然后推断一个较大的时间保证稳定的测量结果。



### 5. 常用测试用法

##### 5.1 测试单个文件和单个方法

+ 测试单个文件 go test -v  file_test.go

+ 测试单个函数：go test -v file_test.go -test.run TestFunc

##### 5.2 测试goroutine 是否竞争

```
go test -race
```

##### 5. 3 TestMain函数

在测试之前或之后进行额外的设置（setup）或拆卸（teardown), 测试进入的第一个函数

```
func TestMain(m *testing.M)
```

##### 5.4 测试覆盖率

```
go tool cover -html=c.out
```



### 6. 参考资料

+ https://github.com/quii/learn-go-with-tests  //非常重要
+ https://github.com/stretchr/testify

+ https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter09/09.0.html

+ https://studygolang.com/articles/12587

