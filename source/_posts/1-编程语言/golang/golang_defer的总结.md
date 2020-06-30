---
title: golang_defer的总结
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: c0441565
date: 2020-02-01 00:00:00
---

每次被 `defer` 坑的死去活来, 今天有时间来整理一下.

<!-- more -->

# 1. 匿名返回值不会修改

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("a return:", a())
}

func a() int {
	var i int
	defer func() {
		i++
		fmt.Println("a defer2:", i)
	}()
	defer func() {
		i++
		fmt.Println("a defer1:", i)
	}()
	return i
}

// 打印结果为 a defer1: 1
// 打印结果为 a defer2: 2
// 打印结果为 a return: 0

```

 a()int 函数的返回值没有被提前声明，其值来自于其他变量的赋值，而defer中修改的也是其他变量（其实该defer根本无法直接访问到返回值），因此函数退出时返回值并没有被修改。



# 2. 有名 ret 方式才会修改

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("b return:", b())
}

func b() (i int) {
	defer func() {
		i++
		fmt.Println("b defer2:", i)
	}()
	defer func() {
		i++
		fmt.Println("b defer1:", i)
	}()
	return i // 或者直接 return 效果相同
}

// 打印结果为 b defer1: 1
// 打印结果为 b defer2: 2
// 打印结果为 b return: 2

```

b()(i int) 函数的返回值被提前声明，这使得defer可以访问该返回值，因此在return赋值返回值 i 之后，defer调用返回值 i 并进行了修改，最后致使return调用RET退出函数后的返回值才会是defer修改过的值。



# 3. 匿名指针返回值也会修改

```go
package main

import (
	"fmt"
)

func main() {
	c := c()
	fmt.Println("c return:", *c, c)
}

func c() *int {
	var i int
	defer func() {
		i++
		fmt.Println("c defer2:", i, &i)
	}()
	defer func() {
		i++
		fmt.Println("c defer1:", i, &i)
	}()
	return &i
}

// 打印结果为 c defer1: 1 0xc082008340
// 打印结果为 c defer2: 2 0xc082008340
// 打印结果为 c return: 2 0xc082008340

```

虽然 c()int 的返回值没有被提前声明，但是由于 c()*int 的返回值是指针变量，那么在return将变量 i 的地址赋给返回值后，defer再次修改了 i 在内存中的实际值，因此return调用RET退出函数时返回值虽然依旧是原来的指针地址，但是其指向的内存实际值已经被成功修改了。



# 4. defer 推迟的仅仅是函数体

defer声明时会先计算确定参数的值，defer推迟执行的仅是其函数体。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	defer P(time.Now())
	time.Sleep(1e9) // 1s
	fmt.Println("1", time.Now())
}
func P(t time.Time) {
	fmt.Println("2", t) // 此处还是老的值
	fmt.Println("3", time.Now())
}

/*
1 2020-06-30 20:27:58.893326 +0800 CST m=+1.000510368
2 2020-06-30 20:27:57.892902 +0800 CST m=+0.000117140
3 2020-06-30 20:27:58.89402 +0800 CST m=+1.001204598
*/

```



# 5. 先 defer 再 panic

```go
package main

import "fmt"

func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	a := 1
	b := 2
	defer calc("1", a, calc("10", a, b))
	a = 0
	defer calc("2", a, calc("20", a, b))
	b = 1
	panic("触发异常")
}

/*
10 1 2 3
20 0 2 2
2 0 2 2
1 1 3 4
panic: 触发异常
*/

```



# 6. 调用os.Exit时defer不会被执行

当发生panic时，所在goroutine的所有defer会被执行，但是当调用os.Exit()方法退出程序时，defer并不会被执行。

```go
package main

import (
	"fmt"
	"os"
)

func deferExit() {
	defer func() {
		fmt.Println("defer")
	}()
	os.Exit(0)
}
func main() {
	deferExit() // 什么也不输出
}
```





# 7. 总结

1.  多个defer的执行顺序为“后进先出”； 
2.  所有函数在执行RET返回指令之前，都会先检查是否存在defer语句，若存在则先逆序调用defer语句进行收尾工作再退出返回； 
3.  匿名返回值是在return执行时被声明，有名返回值则是在函数声明的同时被声明，因此在defer语句中只能访问有名返回值，而不能直接访问匿名返回值； 
4.  return其实应该包含前后两个步骤：
   + 第一步是给返回值赋值（若为有名返回值则直接赋值，若为匿名返回值则先声明再赋值）；
   + 第二步是调用RET返回指令并传入返回值，而RET则会检查defer是否存在，若存在就先逆序插播defer语句，最后RET携带返回值退出函数；

‍‍因此，‍‍defer、return、返回值三者的执行顺序应该是：return最先给返回值赋值；接着defer开始执行一些收尾工作；最后RET指令携带返回值退出函数。



# 8. 脑力风暴

+ 顺序:  defer声明抓取看到的值,   return赋值,  defer后进先出,  带着返回值退出
+ defer 可以访问有名返回值, 无法访问匿名返回值(压栈时看不到)