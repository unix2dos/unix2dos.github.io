---
title: 'golang_struct_interface嵌套传参和多态'
tags: golang
---


### interface struct 嵌套

1. struct struct //继承(不能多态), 如果内部struct实现了接口, 它也相当于实现了接口
2. struct interface //可以多态
3. interface interface  //单纯的导入
4. interface struct  //不允许


<!-- more -->

### struct定义怎么都行, 有interface{}参与struct,就有限制

#### struct方法参数是指针还是值 (此处是单纯的结构, 定义传递什么都行)

无论方法参数定义成指针还是值, 都可以调用

```
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
	s := new(S)
	s.Point(1)
	s.Value()
	fmt.Printf("%T\n", s)

	v := S{}
	v.Point(2)
	v.Value()			//此处是重点, 竟然这个也自动转换
	fmt.Printf("%T\n", v)

	p := &v
	p.Point(3)
	p.Value()
	fmt.Printf("%T\n", p)
}
//output:
1
*main.S
2
main.S
3
*main.S
```


#### interface 实现struct方法参数是指针还是值


接受者是值的都可以赋值

```
type I interface {
	Get()
}
type S struct {
}

func (s S) Get() {
	fmt.Println("get")
}

func main() {
	ss := S{}

	var i I
	i = ss
	i.Get()

	i = &ss
	i.Get()
}
```

接受者是指针的只能指针赋值

```
type I interface {
	Get()
}
type S struct {
}

func (s *S) Get() {
	fmt.Println("get")
}

func main() {
	ss := S{}

	var i I
	//i = ss , 此处编译不过
	//i.Get()

	i = &ss
	i.Get()
}
```


### golang多态实现

```
type P interface {
	Say()
}
type P1 struct{}
type P2 struct{}

func (p *P1) Say() { fmt.Println("say p1") }
func (p *P2) Say() { fmt.Println("say p2") }


func main() {
	p1 := &P1{}
	p2 := &P2{}
	var p P
	p = p1
	p.Say()
	p = p2
	p.Say()
}
```