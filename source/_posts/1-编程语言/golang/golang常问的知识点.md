---
title: golang常问的知识点
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: 19ae52d4
date: 2019-03-01 00:00:05
---



# 1. channel

### 1.1 关闭有缓冲数据的 channel, 还能读取吗

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



### 1.2 channel 被close后，还能被接收或 select 吗

+ 如果有缓存，值能被收到，ok 是 true

+ 如果无缓存，值是0，ok 是 false，但不会 panic
+ 关闭后, 就是你可以读, 但是不能写



### 1.3 如何判断channel已经关闭了 ？

+ Go 中没有一个內建函数去检测管道是否已经被关闭.
+ 直接读取channel结构hchan的closed字段, 不安全

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

+ 正常做法是要监听 close()的 channel,  收到通知

  如果您不知道通道是否关闭并且盲目地写入该通道，则说明您的程序设计不良。 重新设计它，使其在关闭后无法写入。

+ 可用 bool值，但是会多读一个值

  ```go
  value, ok := <- channel 
  if !ok {    
    // channel was closed and drained 
  }
  ```




### 1.4 channel 的零值

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



### 1.5 优雅的关闭 channel

https://www.liuvv.com/p/8b210700.html



### 1.6 什么时候用 channel， 什么时候用Mutex

https://github.com/golang/go/wiki/MutexOrChannel

channel的能力是让数据流动起来，擅长的是数据流动的场景

+ 传递数据的所有权，即把某个数据发送给其他协程

+ 分发任务，每个任务都是一个数据

+ 交流异步结果，结果是一个数据

mutex的能力是数据不动，某段时间只给一个协程访问数据的权限擅长数据位置固定的场景

+ 缓存

+ 状态




# 2. 数据结构

### 2.1 map 为什么无序

+ map扩容, key 会移动

  map 在扩容后，会发生 key 的搬迁，原来落在同一个 bucket 中的 key，搬迁后，有些 key 就要远走高飞了（bucket 序号加上了 2^B）。而遍历的过程，就是按顺序遍历 bucket，同时按顺序遍历 bucket 中的 key。搬迁后，key 的位置发生了重大的变化，有些 key 飞上高枝，有些 key 则原地不动。这样，遍历 map 的结果就不可能按原来的顺序了。

+ 底层代码随机值遍历

  Go 做得更绝，当我们在遍历 map 时，并不是固定地从 0 号 bucket 开始遍历，每次都是从一个随机值序号的 bucket 开始遍历，并且是从这个 bucket 的一个随机序号的 cell 开始遍历。这样，即使你是一个写死的 map，仅仅只是遍历它，也不太可能会返回一个固定序列的 key/value 对了。



### 2.2 map 的key 可以是什么类型

map中的key可以是任何的类型，只要它的值能比较是否相等. Go语言里是无法重载操作符的

- 布尔值

- 数字

- 字符串

- 指针

  Pointer values are comparable. Two pointer values are equal if they point to the same variable or if both have value nil. Pointers to distinct zero-size variables may or may not be equal.

  当指针指向同一变量，或同为nil时指针相等，但指针指向不同的零值时可能不相等。

- Channel

  Channel values are comparable. Two channel values are equal if they were created by the same call to make or if both have value nil.

  Channel当指向同一个make创建的或同为nil时才相等。

- Interface

  Interface values are comparable. Two interface values are equal if they have identical dynamic types and equal dynamic values or if both have value nil.

  当接口有相同的动态类型并且有相同的动态值，或者值为都为nil时相等。

- 结构体

  Struct values are comparable if all their fields are comparable. Two struct values are equal if their corresponding non-blank fields are equal.

  结构体当所有字段的值相同，并且没有相应的非空白字段时，则他们相等。

- 只包含上述类型的数组。

  Array values are comparable if values of the array element type are comparable. Two array values are equal if their corresponding elements are equal.

  两个数组只要他们包括的元素，每个元素的值相同，则他们相等。

但不能是：

- slice
- map
- function



### 2.3 map 如何有序遍历

+ map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序。

+ 有序遍历

  如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序（使用 sort 包），然后可以使用切片的 for-range 方法打印出所有的 key 和 value。




### 2.4 slice 和 map 并发安全吗

map和slice都是并发不安全的, 解决方案:

+ 加锁
+ 使用 channel 串行化
+ sync.Map



### 2.5 slice 底层原理

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



# 5. interface

### 5.1 interface 当参数时的坑, 传递 nil 也不是 nil

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



# 6. make

### 6.1 make时传递的数字参数

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



### 6.2 make 和 new的区别

+ make 只能用于 slice,map,channel

  返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型是引用类型。

+ new(T) 返回 T 的指针 *T 并指向 T 的零值。



# 7. 函数

### 7.1 函数参数传递，是值还是引用

​	Go 中函数传参仅有值传递一种方式, 只有slice, map, channel 本身是引用类型, 所以可以改

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

   


# 8. 其他问题

### 8.1 字节和字节对齐

+ 字节数 

  + 64位系统下, int 默认 int64 ,  8个字节, 

  + float 64,  8个字节

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

	var f string
	fmt.Println(unsafe.Sizeof(f)) //16

	var g chan int
	fmt.Println(unsafe.Sizeof(g)) //8

	var h []int
	fmt.Println(unsafe.Sizeof(h)) //24

	var i map[int]int
	fmt.Println(unsafe.Sizeof(i)) //8
}
```

+ 字节对齐(根据操作系统的位数对齐的)

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

type Info6 struct {
	a bool   //1
	b string //16
	c bool   //1
}

type Info7 struct {
	a bool   //1
	c bool   //1
	b string //16
}

func main() {
	fmt.Println(unsafe.Sizeof(Info1{})) //2
	fmt.Println(unsafe.Sizeof(Info2{})) //16(8+8)
	fmt.Println(unsafe.Sizeof(Info3{})) //8(4+4)
	fmt.Println(unsafe.Sizeof(Info4{})) //24(8+8+8)
	fmt.Println(unsafe.Sizeof(Info5{})) //16(1+1+8 = 8+8)
	fmt.Println(unsafe.Sizeof(Info6{})) //32(1+16+1 = 8+16+8)
	fmt.Println(unsafe.Sizeof(Info7{})) //16(1+1+16 = 8+16)
}
```



### 8.2 非运行多态

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



# 9. 引用和传值

Go语言是没有引用传递的, Go里只有传值（值传递）。

slice / map / chan 是golang的3个引用类型， 本质上 它本身/或者它的一个成员 是指针

+ map

  把 map本身想象成指针, 看指针的值不一样

```go
func main() {
	persons := map[string]int{
		"张三": 19,
	}
	fmt.Println("map old:", persons)
	fmt.Printf("原始map的内存地址是：%p\n", &persons)
	modify(persons)
	fmt.Println("map new:", persons)
}

func modify(p map[string]int) {
	fmt.Printf("函数里接收到map的内存地址是：%p\n", &p)
	p["张三"] = 20
}


/*
map old: map[张三:19]

原始map的内存地址是：0xc000100018
函数里接收到map的内存地址是：0xc000100028

map new: map[张三:20]

*/
```

+ Slice

  通过下标可以直接修改, 但是 append 指针变了, 需要传出来才可以

```go
func main() {
	ages := []int{1, 2, 3}
	fmt.Println("ages old:", ages)
	fmt.Printf("原始slice的头内存地址%p  指针:%p\n", ages, &ages)
	modify(ages)
	fmt.Println("ages new:", ages)
}

func modify(ages []int) {
	fmt.Printf("函数里的头内存地址%p   指针:%p\n", ages, &ages)
	ages[0] = 0
	ages = append(ages, 4, 5, 6, 7)
	fmt.Printf("函数里的 append 头内存地址%p  指针:%p\n", ages, &ages)
}


/*
ages old: [1 2 3]

原始slice的头内存地址		   0xc00011c000       			  指针:0xc00011a000
函数里的头内存地址         0xc00011c000 			         指针:0xc00011a060 (内部的指针地址,意义不大)
函数里的 append 头内存地址 0xc000120040  			  	   指针:0xc00011a060 (内部的指针地址,意义不大)

ages new: [0 2 3]

*/
```



# 10. slice

### 10.1 分配内存

https://github.com/golang/go/blob/296ddf2a936a30866303a64d49bc0e3e034730a8/src/runtime/slice.go#L186

slice伸缩容的时候 一般是按照2倍扩容，当oldcap>=1024， 会在原cap上每次递增25%来扩容；

 扩容时，底层结构体ptr指向的数组放生变化

```go
/*
1. 可以看出, cap 是2倍增长的
2. ss 的指针变了
3. &ss 的指针没有变, &ss操作返回的是该切片的 内部内部内部  指针地址,这个“指针地址" 本身地址肯定不变, 只是它指的值变了
*/

func main() {
	ss := []int{}
	for i := 1; i < 20; i++ {
		fmt.Printf("addr:%p %p,len:%d,cap:%d\n", ss, &ss, len(ss), cap(ss))
		ss = append(ss, i)
	}
}

/*
addr:0x11aac78    0xc0000a6020,len:0,cap:0

addr:0xc0000b4020 0xc0000a6020,len:1,cap:1

addr:0xc0000b4040 0xc0000a6020,len:2,cap:2

addr:0xc0000b6020 0xc0000a6020,len:3,cap:4
addr:0xc0000b6020 0xc0000a6020,len:4,cap:4

addr:0xc0000b8040 0xc0000a6020,len:5,cap:8
addr:0xc0000b8040 0xc0000a6020,len:6,cap:8
addr:0xc0000b8040 0xc0000a6020,len:7,cap:8
addr:0xc0000b8040 0xc0000a6020,len:8,cap:8

addr:0xc0000ba000 0xc0000a6020,len:9,cap:16
addr:0xc0000ba000 0xc0000a6020,len:10,cap:16
addr:0xc0000ba000 0xc0000a6020,len:11,cap:16
addr:0xc0000ba000 0xc0000a6020,len:12,cap:16
addr:0xc0000ba000 0xc0000a6020,len:13,cap:16
addr:0xc0000ba000 0xc0000a6020,len:14,cap:16
addr:0xc0000ba000 0xc0000a6020,len:15,cap:16
addr:0xc0000ba000 0xc0000a6020,len:16,cap:16

addr:0xc0000bc000 0xc0000a6020,len:17,cap:32
addr:0xc0000bc000 0xc0000a6020,len:18,cap:32

*/
```

