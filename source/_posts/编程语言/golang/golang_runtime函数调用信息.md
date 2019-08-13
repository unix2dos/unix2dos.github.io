---
title: golang_runtime函数调用信息
tags:
  - golang
abbrlink: 375281af
categories:
  - 编程语言
  - golang
date: 2018-07-07 14:19:52
---


函数的调用信息是程序中比较重要运行期信息, 在很多场合都会用到(比如调试或日志)。

Go 语言 `runtime` 包的 `runtime.Caller` / `runtime.Callers` / `runtime.FuncForPC` 等几个函数提供了获取函数调用者信息的方法.

 

这几个函数的文档链接:

- <http://golang.org/pkg/runtime/#Caller>

- <http://golang.org/pkg/runtime/#Callers>

- <http://golang.org/pkg/runtime/#FuncForPC>

  

## runtime.Caller的用法(常用)

函数的签名如下:

```
func runtime.Caller(skip int) (pc uintptr, file string, line int, ok bool)
```

`runtime.Caller` 返回当前 `goroutine` 的栈上的函数调用信息. 主要有当前的`pc` 值和调用的文件和行号等信息. 若无法获得信息, 返回的 `ok` 值为 `false`.

 <!-- more -->

其输入参数 `skip` 为要跳过的栈帧数, 若为 `0` 则表示 `runtime.Caller` 的调用者.

*注意:由于历史原因, runtime.Caller 和 runtime.Callers 中的 skip 含义并不相同, 后面会讲到.*



下面是一个简单的例子, 打印函数调用的栈帧信息:

```
func main() {
	for skip := 0; skip < 3; skip++ {
		pc, file, line, ok := runtime.Caller(skip)
		if !ok {
			break
		}
		fmt.Printf("skip = %v, pc = %v, file = %v, line = %v\n", skip, pc, file, line)
	}
	// Output:
	// skip = 0, pc = 4198453, file = caller.go, line = 10
	// skip = 1, pc = 4280066, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 220
	// skip = 2, pc = 4289712, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
}
```



## runtime.Callers的用法

函数的签名如下:

```
func runtime.Callers(skip int, pc []uintptr) int
```

`runtime.Callers` 函数和 `runtime.Caller` 函数虽然名字相似(多一个后缀`s`), 但是函数的参数/返回值和参数的意义都有很大的差异.



`runtime.Callers` 把调用它的函数Go程栈上的程序计数器填入切片 `pc` 中. 参数 `skip` 为开始在 pc 中记录之前所要跳过的栈帧数, **若为 0 则表示 runtime.Callers 自身的栈帧, 若为 1 则表示调用者的栈帧**. 该函数返回写入到 `pc` 切片中的项数(受切片的容量限制).



下面是 `runtime.Callers` 的例子, 用于输出每个栈帧的 `pc` 信息:

```
func main() {
	pc := make([]uintptr, 1024)
	for skip := 0; ; skip++ {
		n := runtime.Callers(skip, pc)
		if n <= 0 {
			break
		}
		fmt.Printf("skip = %v, pc = %v\n", skip, pc[:n])
	}
	// Output:
	// skip = 0, pc = [4304486 4198562 4280114 4289760]
	// skip = 1, pc = [4198562 4280114 4289760]
	// skip = 2, pc = [4280114 4289760]
	// skip = 3, pc = [4289760]
}
```

输出新的 `pc` 长度和 `skip` 大小有逆相关性. `skip = 0` 为 `runtime.Callers` 自身的信息.

这个例子比前一个例子多输出了一个栈帧, 就是因为多了一个 `runtime.Callers` 栈帧的信息 (前一个例子是没有 `runtime.Caller` 信息的(*注意:没有 s 后缀*)).



## runtime.Callers 和 runtime.Caller 的异同

因为前面2个例子为不同的程序, 输出的 `pc` 值并不具备参考性. 现在我们看看在同一个例子的输出结果如何:

 

```
func main() {
	for skip := 0; ; skip++ {
		pc, file, line, ok := runtime.Caller(skip)
		if !ok {
			break
		}
		fmt.Printf("skip = %v, pc = %v, file = %v, line = %v\n", skip, pc, file, line)
	}
	// Output:
	// skip = 0, pc = 4198456, file = caller.go, line = 10
	// skip = 1, pc = 4280962, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 220
	// skip = 2, pc = 4290608, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
	pc := make([]uintptr, 1024)
	for skip := 0; ; skip++ {
		n := runtime.Callers(skip, pc)
		if n <= 0 {
			break
		}
		fmt.Printf("skip = %v, pc = %v\n", skip, pc[:n])
	}
	// Output:
	// skip = 0, pc = [4305334 4198635 4280962 4290608]
	// skip = 1, pc = [4198635 4280962 4290608]
	// skip = 2, pc = [4280962 4290608]
	// skip = 3, pc = [4290608]
}
```

比如输出结果可以发现, `4280962` 和 `4290608` 两个 `pc` 值是相同的. 它们分别对应 `runtime.main` 和 `runtime.goexit` 函数.



`runtime.Caller` 输出的 `4198456` 和 `runtime.Callers` 输出的 `4198635` 并不相同. 这是因为, 这两个函数的调用位置并不相同, 因此导致了 `pc` 值也不完全相同.



最后就是 `runtime.Callers` 多输出一个 `4305334` 值, 对应`runtime.Callers`内部的调用位置.

由于Go语言(Go1.2)采用分段堆栈, 因此不同的 `pc` 之间的大小关系并不明显.



## runtime.FuncForPC 的用途

函数的签名如下:

```
func runtime.FuncForPC(pc uintptr) *runtime.Func
func (f *runtime.Func) FileLine(pc uintptr) (file string, line int)
func (f *runtime.Func) Entry() uintptr
func (f *runtime.Func) Name() string
```



其中 `runtime.FuncForPC` 返回包含给定 `pc` 地址的函数, 如果是无效 `pc` 则返回 `nil` .

`runtime.Func.FileLine` 返回与 `pc` 对应的源码文件名和行号. 安装文档的说明, 如果`pc`不在函数帧范围内, 则结果是不确定的.

`runtime.Func.Entry` 对应函数的地址. 

`runtime.Func.Name` 返回该函数的名称. 



```
func main() {
	for skip := 0; ; skip++ {
		pc, _, _, ok := runtime.Caller(skip)
		if !ok {
			break
		}
		p := runtime.FuncForPC(pc)
		file, line := p.FileLine(0)

		fmt.Printf("skip = %v, pc = %v\n", skip, pc)
		fmt.Printf("  file = %v, line = %d\n", file, line)
		fmt.Printf("  entry = %v\n", p.Entry())
		fmt.Printf("  name = %v\n", p.Name())
	}
	// Output:
	// skip = 0, pc = 4198456
	//   file = caller.go, line = 8
	//   entry = 4198400
	//   name = main.main
	// skip = 1, pc = 4282882
	//   file = $(GOROOT)/src/pkg/runtime/proc.c, line = 179
	//   entry = 4282576
	//   name = runtime.main
	// skip = 2, pc = 4292528
	//   file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
	//   entry = 4292528
	//   name = runtime.goexit
	
	
	pc := make([]uintptr, 1024)
	for skip := 0; ; skip++ {
		n := runtime.Callers(skip, pc)
		if n <= 0 {
			break
		}
		fmt.Printf("skip = %v, pc = %v\n", skip, pc[:n])
		for j := 0; j < n; j++ {
			p := runtime.FuncForPC(pc[j])
			file, line := p.FileLine(0)

			fmt.Printf("  skip = %v, pc = %v\n", skip, pc[j])
			fmt.Printf("    file = %v, line = %d\n", file, line)
			fmt.Printf("    entry = %v\n", p.Entry())
			fmt.Printf("    name = %v\n", p.Name())
		}
		break
	}
	// Output:
	// skip = 0, pc = [4307254 4198586 4282882 4292528]
	//   skip = 0, pc = 4307254
	//     file = $(GOROOT)/src/pkg/runtime/runtime.c, line = 315
	//     entry = 4307168
	//     name = runtime.Callers
	//   skip = 0, pc = 4198586
	//     file = caller.go, line = 8
	//     entry = 4198400
	//     name = main.main
	//   skip = 0, pc = 4282882
	//     file = $(GOROOT)/src/pkg/runtime/proc.c, line = 179
	//     entry = 4282576
	//     name = runtime.main
	//   skip = 0, pc = 4292528
	//     file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
	//     entry = 4292528
	//     name = runtime.goexit
}
```



根据测试, 如果是无效 `pc` (比如`0`), `runtime.Func.FileLine` 一般会输出当前函数的开始行号. 不过在实践中, 一般会用 `runtime.Caller` 获取文件名和行号信息, `runtime.Func.FileLine` 很少用到.



## 定制的 CallerName 函数

基于前面的几个函数, 我们可以方便的定制一个 `CallerName` 函数. 函数 `CallerName` 返回调用者的函数名/文件名/行号等用户友好的信息.



```
func CallerName(skip int) (name, file string, line int, ok bool) {
	var pc uintptr
	if pc, file, line, ok = runtime.Caller(skip + 1); !ok {
		return
	}
	name = runtime.FuncForPC(pc).Name()
	return
}
```

其中在执行 `runtime.Caller` 调用时, 参数 `skip + 1` 用于抵消 `CallerName` 函数自身的调用.



```
func main() {
	for skip := 0; ; skip++ {
		name, file, line, ok := CallerName(skip)
		if !ok {
			break
		}
		fmt.Printf("skip = %v\n", skip)
		fmt.Printf("  file = %v, line = %d\n", file, line)
		fmt.Printf("  name = %v\n", name)
	}
	// Output:
	// skip = 0
	//   file = caller.go, line = 19
	//   name = main.main
	// skip = 1
	//   file = C:/go/go-tip/src/pkg/runtime/proc.c, line = 220
	//   name = runtime.main
	// skip = 2
	//   file = C:/go/go-tip/src/pkg/runtime/proc.c, line = 1394
	//   name = runtime.goexit
}
```



## Go 语言中函数的类型

在 Go 语言中, 除了语言定义的普通函数调用外, 还有闭包函数/init函数/全局变量初始化等不同的函数调用类型.

为了便于测试不同类型的函数调用, 我们包装一个 `PrintCallerName` 函数. 该函数用于输出调用者的信息.

```
func PrintCallerName(skip int, comment string) bool {
	name, file, line, ok := CallerName(skip + 1)
	if !ok {
		return false
	}
	fmt.Printf("skip = %v, comment = %s\n", skip, comment)
	fmt.Printf("  file = %v, line = %d\n", file, line)
	fmt.Printf("  name = %v\n", name)
	return true
}
```



然后编写以下的测试代码(函数闭包调用/全局变量初始化/init函数等):

```
var a = PrintCallerName(0, "main.a")
var b = PrintCallerName(0, "main.b")

func init() {
	a = PrintCallerName(0, "main.init.a")
}

func init() {
	b = PrintCallerName(0, "main.init.b")
	func() {
		b = PrintCallerName(0, "main.init.b[1]")
	}()
}

func main() {
	a = PrintCallerName(0, "main.main.a")
	b = PrintCallerName(0, "main.main.b")
	func() {
		b = PrintCallerName(0, "main.main.b[1]")
		func() {
			b = PrintCallerName(0, "main.main.b[1][1]")
		}()
		b = PrintCallerName(0, "main.main.b[2]")
	}()
}
```

输出结果如下:

```
// Output:
// skip = 0, comment = main.a
//   file = caller.go, line = 8
//   name = main.init
// skip = 0, comment = main.b
//   file = caller.go, line = 9
//   name = main.init
// skip = 0, comment = main.init.a
//   file = caller.go, line = 12
//   name = main.init·1
// skip = 0, comment = main.init.b
//   file = caller.go, line = 16
//   name = main.init·2
// skip = 0, comment = main.init.b[1]
//   file = caller.go, line = 18
//   name = main.func·001
// skip = 0, comment = main.main.a
//   file = caller.go, line = 23
//   name = main.main
// skip = 0, comment = main.main.b
//   file = caller.go, line = 24
//   name = main.main
// skip = 0, comment = main.main.b[1]
//   file = caller.go, line = 26
//   name = main.func·003
// skip = 0, comment = main.main.b[1][1]
//   file = caller.go, line = 28
//   name = main.func·002
// skip = 0, comment = main.main.b[2]
//   file = caller.go, line = 30
//   name = main.func·003
```



观察输出结果, 可以发现以下几个规律:

- 全局变量的初始化调用者为 `main.init` 函数
- 自定义的 `init` 函数有一个数字后缀, 根据出现的顺序进编号. 比如 `main.init·1` 和 `main.init·2` 等.
- 闭包函数采用 `main.func·001` 格式命名, 安装闭包定义结束的位置顺序进编号.

 

## 不同 Go 程序启动流程

基于函数调用者信息可以很容易的验证各种环境的程序启动流程.

我们需要建立一个独立的 `caller` 目录, 里面有三个测试代码.

`caller/main.go` 主程序:

```
package main

import (
	"fmt"
	"regexp"
	"runtime"
)

func main() {
	_ = PrintCallerName(0, "main.main._")
}

func PrintCallerName(skip int, comment string) bool {
	// 实现和前面的例子相同
}

func CallerName(skip int) (name, file string, line int, ok bool) {
	// 实现和前面的例子相同
}
```

`caller/main_test.go` 主程序的测试文件(同在一个`main`包):

```
package main

import (
	"fmt"
	"testing"
)

func TestPrintCallerName(t *testing.T) {
	for skip := 0; ; skip++ {
		name, file, line, ok := CallerName(skip)
		if !ok {
			break
		}
		fmt.Printf("skip = %v, name = %v, file = %v, line = %v\n", skip, name, file, line)
	}
	t.Fail()
}
```

`caller/example_test.go` 主程序的包的调用者(在新的`main_test`包):

```
package main_test

import (
	myMain "."
	"fmt"
)

func Example() {
	for skip := 0; ; skip++ {
		name, file, line, ok := myMain.CallerName(skip)
		if !ok {
			break
		}
		fmt.Printf("skip = %v, name = %v, file = %v, line = %v\n", skip, name, file, line)
	}
	// Output: ?
}
```

然后进入 `caller` 目录, 运行 `go run test` 可以得到以下的输出结果:

```
skip = 0, name = caller.TestPrintCallerName, file = caller/main_test.go, line = 10
skip = 1, name = testing.tRunner, file = $(GOROOT)/src/pkg/testing/testing.go, line = 391
skip = 2, name = runtime.goexit, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
--- FAIL: TestPrintCallerName (0.00 seconds)
--- FAIL: Example (2.0001ms)
got:
skip = 0, name = caller_test.Example, file = caller/example_test.go, line = 10

skip = 1, name = testing.runExample, file = $(GOROOT)/src/pkg/testing/example.go, line = 98
skip = 2, name = testing.RunExamples, file = $(GOROOT)/src/pkg/testing/example.go, line = 36
skip = 3, name = testing.Main, file = $(GOROOT)/src/pkg/testing/testing.go, line = 404
skip = 4, name = main.main, file = $(TEMP)/go-build365033523/caller/_test/_testmain.go, line = 51
skip = 5, name = runtime.main, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 220
skip = 6, name = runtime.goexit, file = $(GOROOT)/src/pkg/runtime/proc.c, line = 1394
want:
?
FAIL
exit status 1
FAIL    caller        0.254s
```

分析输出数据我们可以发现, 测试代码和例子代码的启动流程和普通的程序流程都不太一样.

`测试代码`的启动流程:

1. `runtime.goexit` 还是入口
2. 但是 `runtime.goexit` 不在调用 `runtime.main` 函数, 而是调用 `testing.tRunner` 函数
3. `testing.tRunner` 函数由 `go test` 命令生成, 用于执行各个测试函数

`例子代码`的启动流程:

1. `runtime.goexit` 还是入口
2. 然后 `runtime.goexit` 调用 `runtime.main` 函数
3. 最终 `runtime.main` **调用go test 命令生成的 main.main 函数**, 在 `_test/_testmain.go` 文件
4. 然后调用 `testing.Main`, 改函数执行各个例子函数

另外, 从这个例子我们可以发现, 我们自己写的 `main.main` 函数所在的 `main` 包也 可以被其他包导入. 但是其他包导入之后的 `main` 包里的 `main` 函数就不再是 `main.main` 函数了. 因此, 程序的入口也就不是自己写的 `main.main` 函数了.



## 总结

Go 语言 `runtime` 包的 `runtime.Caller` / `runtime.Callers` / `runtime.FuncForPC` 等函数虽然看起来比较简单, 但是功能却非常强大.

这几个函数不仅可以解决一些实际的工程问题 , 而且非常适合用于调试和分析各种Go程序的运行时信息.
