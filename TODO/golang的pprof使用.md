---
title: "golang的pprof使用"
date: 2020-02-07 00:00:00
tags:
- golang
---

# 1. quick start

```go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
)

func main() {
	log.Println(http.ListenAndServe("localhost:6060", nil))
}

```

<!-- more -->

打开浏览器, 输入 `http://localhost:6060/debug/pprof/`, 内容显示如下:

```bash
/debug/pprof/

Types of profiles available:
Count	Profile
1	allocs
0	block
0	cmdline
4	goroutine
1	heap
0	mutex
0	profile
5	threadcreate
0	trace
full goroutine stack dump
```

上面每一个都是一个超链接, 可以点进去, 看到堆栈信息

![1](golang的pprof使用/0.png)

# 2. 火焰图

+ 火焰图mac需要安装graphviz

    ```bash
    brew install graphviz
    
    # 一直下载不成功, 修改brew源
    
    ```

+ 查看内存使用情况

  ```bash
  go tool pprof http://localhost:6060/debug/pprof/heap
  # 进入如下gdb交互模式:
  
  # top 查看前10个的内存分配情况
  (pprof) top
  Showing nodes accounting for 1.16MB, 100% of 1.16MB total
        flat  flat%   sum%        cum   cum%
      1.16MB   100%   100%     1.16MB   100%  runtime/pprof.writeGoroutineStacks
           0     0%   100%     1.16MB   100%  net/http.(*ServeMux).ServeHTTP
           0     0%   100%     1.16MB   100%  net/http.(*conn).serve
           0     0%   100%     1.16MB   100%  net/http.HandlerFunc.ServeHTTP
           0     0%   100%     1.16MB   100%  net/http.serverHandler.ServeHTTP
           0     0%   100%     1.16MB   100%  net/http/pprof.Index
           0     0%   100%     1.16MB   100%  net/http/pprof.handler.ServeHTTP
           0     0%   100%     1.16MB   100%  runtime/pprof.(*Profile).WriteTo
           0     0%   100%     1.16MB   100%  runtime/pprof.writeGoroutine
           
           
  # tree 以树状显示
  (pprof) tree
  Showing nodes accounting for 1.16MB, 100% of 1.16MB total
  ----------------------------------------------------------+-------------
        flat  flat%   sum%        cum   cum%   calls calls% + context
  ----------------------------------------------------------+-------------
                                              1.16MB   100% |   runtime/pprof.writeGoroutine
      1.16MB   100%   100%     1.16MB   100%                | runtime/pprof.writeGoroutineStacks
      
      
  # png 以图片格式输出
  
  # svg 生成浏览器可以识别的svg文件
  ```
  
  
  
  | 列名  | 含义                                                         |
  | ----- | ------------------------------------------------------------ |
  | flat  | 本函数的执行耗时                                             |
  | flat% | flat 占 CPU 总时间的比例。程序总耗时 16.22s, Eat 的 16.19s 占了 99.82% |
  | sum%  | 前面每一行的 flat 占比总和                                   |
  | cum   | 累计量。指该函数加上该函数调用的函数总耗时                   |
  | cum%  | cum 占 CPU 总时间的比例                                      |
  
+ 查看 CPU

  ```bash
  go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
  
  #进入gdb交互模式
  ```

  

# 2. 使用






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



go tool pprof http://127.0.0.1:8080/debug/pprof/profile



然后等待一会儿，出现如下提示后 (pprof) 输入 svg 在当前指令执行目录生成 .svg 文件打开即可。也可以执行 web，但是要有浏览器会直接生成并用浏览器打开文件，像服务器这种没法装浏览器的只能执行 svg 了







# 4. 参考资料



+ https://golang.org/pkg/net/http/pprof/
+ https://blog.wolfogre.com/posts/go-ppof-practice/



https://zhuanlan.zhihu.com/p/71529062

https://xargin.com/pprof-and-flamegraph/