---
title: golang常见问题
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: 19ae52d4
date: 2019-03-01 00:00:05
---



### 1. 关闭有缓冲数据的 channel发生什么

只有当channel无数据，且channel被close了，才会返回ok=false。 只要有堆积数据，即使 close() 也不会返回关闭状态。

关闭后有数据也能从里面得到数据，除非消耗空了。

<!-- more -->

```go
func main() {

	c := make(chan int, 10)
	c <- 1
	c <- 2
	c <- 3
	close(c)

	for {
		data, ok := <-c
		if !ok {
			return
		}
		fmt.Println(data, ok)
	}
}

/*
1 true
2 true
3 true
*/
```



> 那么如何在channel堆积的情况下，得知channel已经关闭了 ？

可以直接读取channel结构hchan的closed字段，但问题chan.go没有开放这样的api，可以用黑科技,但是他不安全.

```go
import (
    "unsafe"
    "reflect"
)


func isChanClosed(ch interface{}) bool {
    if reflect.TypeOf(ch).Kind() != reflect.Chan {
        panic("only channels!")
    }

    // get interface value pointer, from cgo_export 
    // typedef struct { void *t; void *v; } GoInterface;
    // then get channel real pointer
    cptr := *(*uintptr)(unsafe.Pointer(
        unsafe.Pointer(uintptr(unsafe.Pointer(&ch)) + unsafe.Sizeof(uint(0))),
    ))

    // this function will return true if chan.closed > 0
    // see hchan on https://github.com/golang/go/blob/master/src/runtime/chan.go 
    // type hchan struct {
    // qcount   uint           // total data in the queue
    // dataqsiz uint           // size of the circular queue
    // buf      unsafe.Pointer // points to an array of dataqsiz elements
    // elemsize uint16
    // closed   uint32
    // **

    cptr += unsafe.Sizeof(uint(0))*2
    cptr += unsafe.Sizeof(unsafe.Pointer(uintptr(0)))
    cptr += unsafe.Sizeof(uint16(0))
    return *(*uint32)(unsafe.Pointer(cptr)) > 0
}
```



### 2. 如何判断 channel close 状态

关闭一个已经关闭的管道会触发 panic，所以，关闭者不知道管道是否关闭仍去关闭它，这是一个危险的行为。

发送数据到一个关闭的管道会触发 panic, 所以，发送者不知道管道是否关闭仍去发送消息给它，这是一个危险的行为。

Go 中没有一个內建函数去检测管道是否已经被关闭（没有办法）。如果您不知道通道是否关闭并且盲目地写入该通道，则说明您的程序设计不良。 重新设计它，使其在关闭后无法写入。

+ 可用 bool值，但是会多读一个值

  ```go
  value, ok := <- channel 
  if !ok {    
    // channel was closed and drained 
  }
  ```

+ 可用上面的黑科技



### 3. channel 被close后，还能被接收或 select 到吗

如果有缓存，值能被收到，ok 是 true

如果无缓存，值是0，ok 是 false，但不会 panic



### 4. channel 的零值

channel的零值是nil。也许会让你觉得比较奇怪，nil的channel有时候也是有一些用处的。

+ 因为对一个nil的channel发送和接收操作会永远阻塞
+ 在select语句中操作nil的channel永远都不会被select到。这使得我们可以用nil来激活或者禁用case。

```go
func main() {

	var c chan int
	fmt.Println(c)

	for {
		select {
		case <-c:
			fmt.Println("nerver")
		default:
		}
	}
}


// 输出 nil 之后无线循环，不会输出 nerver
```



### 5. 优雅的关闭 channel

https://www.liuvv.com/p/8b210700.html



### 6. 如何检测 goroutine泄露

+ golang/gops 

+ pprof

  使用Go提供的pprof工具分析是否发生内存泄露。使用 pprof 的 heap 能够获取程序运行时的内存信息，通过对运行的程序多次采样对比，分析出内存的使用情况。
  
  ```go
  package main
  
  import (
  	"flag"
  	"log"
  	"net/http"
  	_ "net/http/pprof"
  	"sync"
  	"time"
  )
  
  func main() {
  	flag.Parse()
  
  	//这里实现了远程获取pprof数据的接口
  	go func() {
  		log.Println(http.ListenAndServe("localhost:6060", nil))
  	}()
  
  	var wg sync.WaitGroup
  	wg.Add(10)
  	for i := 0; i < 10; i++ {
  		go work(&wg)
  	}
  
  	wg.Wait()
  	// Wait to see the global run queue deplete.
  	time.Sleep(3 * time.Second)
  }
  
  func work(wg *sync.WaitGroup) {
  	time.Sleep(time.Second)
  
  	var counter int
  	for i := 0; i < 1e10; i++ {
  		time.Sleep(time.Millisecond * 100)
  		counter++
  	}
  	wg.Done()
  }
  ```
  
  打开浏览器,访问如下:
  
  ```bash
  # 访问 http://localhost:6060/debug/pprof/
  
  /debug/pprof/
  
  Types of profiles available:
  Count	Profile
  2	allocs
  0	block
  0	cmdline
  15	goroutine
  2	heap
  0	mutex
  0	profile
  10	threadcreate
  0	trace
  ```



##### 6.1 goroutine 泄露的防范

- 创建goroutine时就要想好该goroutine该如何结束
- 使用channel时，要考虑到 channel 阻塞时协程可能的行为
- 实现循环语句时注意循环的退出条件，避免死循环



### 7. 什么时候用 channel， 什么时候用Mutex

https://github.com/golang/go/wiki/MutexOrChannel

channel的能力是让数据流动起来，擅长的是数据流动的场景

+ 传递数据的所有权，即把某个数据发送给其他协程

+ 分发任务，每个任务都是一个数据

+ 交流异步结果，结果是一个数据

mutex的能力是数据不动，某段时间只给一个协程访问数据的权限擅长数据位置固定的场景

+ 缓存

+ 状态

  

### 8. 函数参数传递，是值还是引用

1. 为什么说 **slice**、**map**、**channel** 是引用类型？

2. Go中 **slice** 在传入函数时到底是不是引用传递？如果不是，在函数内为什么能修改其值？

   

+ 其实传递的就是值,但是为什么能改内容呢?

  ```go
  func ChangeMap(value map[string]string) {
    fmt.Printf("map内部 %p\n", &value)
    value["age"] = "30"
  }

  func ChangeSlice(value []string) {
    fmt.Printf("slice内部 %p\n", &value)
    value[0] = "haha"
  }

  func main() {
    map1 := make(map[string]string)
    map1["age"] = "21"
    fmt.Printf("map外部 %p\n", &map1)
    fmt.Println(map1["age"])
    ChangeMap(map1)
    fmt.Println(map1["age"])

    slice1 := make([]string, 0)
    slice1 = append(slice1, "hehe")
    fmt.Println(slice1)
    fmt.Printf("slice外部 %p\n", &slice1)
    ChangeSlice(slice1)
    fmt.Println(slice1)
  }

  /*
  map外部 0xc000092018
  21
  map内部 0xc000092028
  30
  
  
  [hehe]
  slice外部 0xc00008a040
  slice内部 0xc00008a080
  [haha]
  */
  ```



+ 普通的类型就是值传递

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	a := 10
  	fmt.Printf("%#v\n", &a) // (*int)(0xc420018080)
  	vFoo(a)
  }
  
  func vFoo(b int) {
  	fmt.Printf("%#v\n", &b) // (*int)(0xc420018090)
  }
  ```

  

+ 总结

  + Go 中函数传参仅有值传递一种方式.
  + 因为使用了 make, 为我们省去了指针的操作，让我们可以更容易的使用**slice**、**map**、**channel**。
  + 查看make slice的源码：func makeslice(et *_type, len, cap int) unsafe.Pointer可以看到slice返回的就是一个指针值，可以在不同指针类型之间做转换。
  + 这里的**slice**、**map**、**channel**可以理解为引用类型(跟c++的不同)，但是记住引用类型不是传引用。

   

  
### 9. slice 底层原理

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

切片的结构体由3部分构成，Pointer 是指向一个数组的指针，len 代表当前切片的长度，cap 是当前切片的容量。cap 总是大于等于 len 的。



- 如果切片的容量小于1024个元素，那么扩容的时候slice的cap就翻番，乘以2；一旦元素个数超过1024个元素，增长因子就变成1.25，即每次增加原来容量的四分之一。
- 如果扩容之后，还没有触及原数组的容量，那么，切片中的指针指向的位置，就还是原数组，如果扩容之后，超过了原数组的容量，那么，Go就会开辟一块新的内存，把原来的值拷贝过来，这种情况丝毫不会影响到原数组。



### 10. map 无序还是有序，如何有序遍历

+ map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序。

+ 有序遍历

  如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序（使用 sort 包），然后可以使用切片的 for-range 方法打印出所有的 key 和 value。

  

### 11. 并发安全 map

+ 不安全,需要加锁 

+ Go 1.9后使用 sync.Map

  ```go
  func main() {
  	a := sync.Map{}
  	a.Store("name", "levon")
  	fmt.Println(a.Load("name")) //levon true
  }
  ```

  

### 12. 字节和字节对齐

+ 字节数

  int 默认 int64 ,  8个字节, float 64,  8个字节

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var a int
	fmt.Println(unsafe.Sizeof(a)) //8

	var b int32
	fmt.Println(unsafe.Sizeof(b)) //4

	var c int64
	fmt.Println(unsafe.Sizeof(c)) //8

	var d float64
	fmt.Println(unsafe.Sizeof(d)) //8

	var e float32
	fmt.Println(unsafe.Sizeof(e)) //4
}
```

+ 字节对齐

```go
package main

import (
	"fmt"
	"unsafe"
)

type Info1 struct {
	sex    bool //1
	status int8 //1
}

type Info2 struct {
	sex bool //1
	age int  //8
}

type Info3 struct {
	sex bool  //1
	age int32 //4
}

type Info4 struct {
	sex    bool //1
	age    int  //8
	status int8 //1
}

type Info5 struct {
	sex    bool //1
	status int8 //1
	age    int  //8
}

func main() {
	fmt.Println(unsafe.Sizeof(Info1{})) //2
	fmt.Println(unsafe.Sizeof(Info2{})) //16(8+8)
	fmt.Println(unsafe.Sizeof(Info3{})) //8(4+4)
	fmt.Println(unsafe.Sizeof(Info4{})) //24(8+8+8)
	fmt.Println(unsafe.Sizeof(Info5{})) //16(1+1+8 = 8+8)
}
```



### 13. 非运行多态

go 语言中，当子类调用父类方法时，“作用域”将进入父类的作用域，看不见子类的方法存在

```go
package main

import "fmt"

type A struct {
}

func (a *A) ShowA() {
	fmt.Println("showA")
	a.ShowB()
}
func (a *A) ShowB() {
	fmt.Println("showB")
}

type B struct {
	A
}

func (b *B) ShowB() {
	fmt.Println("b showB")
}

func main() {
	b := B{}
	b.ShowA()
}

// showA
// showB,  not b showB
```



### 14. make时传递的数字参数

```go
package main

import "fmt"

func main() {
	s := make([]int, 0)
	fmt.Println(s) //[]
	s = append(s, 1, 2, 3)
	fmt.Println(s) //[1 2 3]
}

```

如果传递了个数, 就是有默认值了

```go
package main

import "fmt"

func main() {
	s := make([]int, 5)
	fmt.Println(s) //[0 0 0 0 0]

  s = append(s, 1, 2, 3)
	fmt.Println(s) //[0 0 0 0 0 1 2 3]
}
```



### 15. interface 当参数时的坑, 传递 nil 也不是 nil

如果需要判断, 请用反射 `reflect.ValueOf(i).IsNil()`

```go
package main

import "fmt"

type I interface {
	A() error
}

type T struct {
}

func (t *T) A() error {
	return nil
}

func testInterface(i I) {
	if i == nil {
		fmt.Println("i is nil")
	} else {
		fmt.Println("i is not nil")
	}
}
func main() {
	t := new(T)
	t = nil
	testInterface(t) //i is not nil
}

```



### 16. panic 和 recover

##### 16.0 defer 和 recover 配合, 并且先声明defer

```go
package main

import (
	"fmt"
)

func main() {
	defer func() { // 必须要先声明defer，否则不能捕获到panic异常
		fmt.Println("a")
		if err := recover(); err != nil {
			fmt.Println(err)
		}
		fmt.Println("b")
	}()

	panic("异常信息")

	fmt.Println("c")
}

```



##### 16.1 panic 会一直传递, 导致主 gorouting 奔溃

```go
package main

import (
	"fmt"
	"time"
)

func testPanic2() {
	panic("testPanic panic2")
}

func testPanic1() {
	go testPanic2()
}

func main() {
	fmt.Println("begin")
	go testPanic1()
	for {
		time.Sleep(time.Second)
	}
}


/*
begin
panic: testPanic panic2

goroutine 5 [running]:
main.testPanic2()
*/
```

##### 16.2 外层的 recover 能捕捉里层的 panic吗

不能

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()

	go func() {
		panic("falut1")
	}()

	time.Sleep(time.Second)
}

/*
panic: falut1

goroutine 18 [running]:
main.main.func2()
        /Users/liuwei/golang/src/golangTest/suanfa/main.go:16 +0x39
created by main.main
        /Users/liuwei/golang/src/golangTest/suanfa/main.go:15 +0x71
exit status 2
*/
```



### 17. interface 实现

Go的interface是由两种类型来实现的：`iface`和`eface`。

其中，`iface`表示的是包含方法的interface，而`eface`代表的是不包含方法的interface，

##### 17.1 eface

一共有两个属性构成，一个是类型信息`_type`，一个是数据信息`data`。

其中，`_type`可以认为是Go语言中所有类型的公共描述，Go语言中几乎所有的数据结构都可以抽象成`_type`，是所有类型的表现，可以说是万能类型。

`data`是指向具体数据的指针。

##### 17.2 iface

```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}

type itab struct {
	inter *interfacetype //此属性用于定位到具体interface
	_type *_type //此属性用于定位到具体interface
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}
```



### 18. slice 和 map 都不是并发安全

并发安全也叫线程安全，在并发中出现了数据的丢失，称为并发不安全。map和slice都是并发不安全的

> 解决方案:

+ 加锁
+ 使用 channel 串行化
+ sync.Map

