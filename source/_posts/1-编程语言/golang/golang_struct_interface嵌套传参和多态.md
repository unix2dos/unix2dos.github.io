---
title: golang_struct_interface嵌套传参和多态
tags: golang
abbrlink: a5099912
categories:
  - 1-编程语言
  - golang
date: 2017-09-01 17:54:46
---


# 1. interface struct 能否相互嵌套

1. struct struct //继承(不能多态), 如果内部struct实现了接口, 它也相当于实现了接口
2. struct interface //可以多态
3. interface interface  //单纯的导入
4. interface struct  //不允许

<!-- more -->

# 2. struct方法参数定义指针还是值 

无论方法参数定义成指针还是值, 都可以调用

```go
package main

import "fmt"

type S struct {
	age int
}

func (s S) Value() {
	fmt.Println(s.age)
}

func (s *S) Point(age int) {
	s.age = age
}

func main() {

	// 自己是指针, 能够调用一切
	s := new(S)
	s.Point(1)
	s.Value()      //1
	fmt.Println(s) //&{1}

	// 自己不是指针,也能调用指针函数修改值
	v := S{}
	v.Point(2)
	v.Value()      //2
	fmt.Println(v) //{2}
}
```



# 3. interface 能否赋值实现了的 struct

### 3.1 struct是值的都可以赋值

```go
package main

import "fmt"

type I interface {
	Get()
}
type S struct {
}

func (s S) Get() {
	fmt.Println("get")
}

func main() {

	var i I

	i = S{}
	i.Get() // get

	i = &S{}
	i.Get() //get
}
```

### 3.2 struct是指针的只能指针赋值

```go
package main

import "fmt"

type I interface {
	Get()
}
type S struct {
}

func (s *S) Get() {
	fmt.Println("get")
}

func main() {

	var i I

	//i = S{} //此处不能赋值
	//i.Get()

	i = &S{}
	i.Get() //get
}
```



# 4. golang假多态


```go
package main

import "fmt"

type P interface {
	Say()
}
type P1 struct{}
type P2 struct{}

func (p *P1) Say() {
	fmt.Println("say p1")
}
func (p *P2) Say() {
	fmt.Println("say p2")
}

func main() {
	p1 := &P1{}
	p2 := &P2{}

	var p P
	p = p1
	p.Say() // say p1
	p = p2
	p.Say() // say p2
}
```



# 5. 非运行时多态

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

