---
title: "golang的pprof使用"
date: 2020-02-07 00:00:00
tags:
- golang
---



<!-- more -->


```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	_ "net/http/pprof"
	"time"
)

func main() {
	go func() {
		for {
			LocalTz()
			doSomething([]byte(`{"a": 1, "b": 2, "c": 3}`))
		}
	}()

	fmt.Println("start api server...")
	panic(http.ListenAndServe(":8080", nil))
}

func doSomething(s []byte) {
	var m map[string]interface{}
	err := json.Unmarshal(s, &m)
	if err != nil {
		panic(err)
	}

	s1 := make([]string, 0)
	s2 := ""
	for i := 0; i < 100; i++ {
		s1 = append(s1, string(s))
		s2 += string(s)
	}
}

func LocalTz() *time.Location {
	tz, _ := time.LoadLocation("Asia/Shanghai")
	return tz
}

```
打开浏览器: http://localhost:8080/debug/pprof/, 输出如下:

```bash
/debug/pprof/

Types of profiles available:
Count	Profile
59	allocs
0	block
0	cmdline
5	goroutine
59	heap
0	mutex
0	profile
9	threadcreate
0	trace
full goroutine stack dump
Profile Descriptions:

allocs: A sampling of all past memory allocations
block: Stack traces that led to blocking on synchronization primitives
cmdline: The command line invocation of the current program
goroutine: Stack traces of all current goroutines
heap: A sampling of memory allocations of live objects. You can specify the gc GET parameter to run GC before taking the heap sample.
mutex: Stack traces of holders of contended mutexes
profile: CPU profile. You can specify the duration in the seconds GET parameter. After you get the profile file, use the go tool pprof command to investigate the profile.
threadcreate: Stack traces that led to the creation of new OS threads
trace: A trace of execution of the current program. You can specify the duration in the seconds GET parameter. After you get the trace file, use the go tool trace command to investigate the trace.
```







go tool pprof http://127.0.0.1:8080/debug/pprof/profile



然后等待一会儿，出现如下提示后 (pprof) 输入 svg 在当前指令执行目录生成 .svg 文件打开即可。也可以执行 web，但是要有浏览器会直接生成并用浏览器打开文件，像服务器这种没法装浏览器的只能执行 svg 了





火焰图需要安装

``` bash
brew install graphviz
```





# 4. 参考资料

![1](golang的pprof使用/0.png)

+ https://zhuanlan.zhihu.com/p/71529062





https://xargin.com/pprof-and-flamegraph/