---
title: golang的一些操作技巧
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: c79ad2a9
date: 2021-01-29 00:00:01
---

好久没写golang相关的blog了, 记录一些常见的golang操作



# 1. 不影响函数调用, 增加参数

先看以下函数调用:

```go
package main

import "fmt"

func ExecUser(name string, age int) {
	fmt.Println("name:", name, "age:", age)
}

func main() {
	ExecUser("levonfly", 9)
}

// name: levonfly age: 9
```

<!-- more -->

这时候我想在原函数参数中传入我的邮箱等信息, 不仅需要修改函数定义, 还会影响以前的调用.

在不影响以前函数调用的情况下, 支持动态传参.  并且在动态的基础上加上 key

```go
package main

import "fmt"

type ArgFunc func() (key string, value string)

func BuildArgFunc(key string, value string) ArgFunc {
	return func() (k string, v string) {
		k = key
		v = value
		return
	}
}

func ExecUser(name string, age int, funcArgs ...ArgFunc) {
	fmt.Println("name:", name, "age:", age)
	for _, funcArg := range funcArgs {
		k, v := funcArg()
		fmt.Println(k, v)
	}
}

func main() {
	ExecUser("levonfly", 9)
	ExecUser("levonfly", 9, BuildArgFunc("email", "levonfly@gmail.com"))
	ExecUser("levonfly", 9, BuildArgFunc("email", "levonfly@gmail.com"), BuildArgFunc("sex", "man"))
}

/*
name: levonfly age: 9

name: levonfly age: 9
email levonfly@gmail.com

name: levonfly age: 9
email levonfly@gmail.com
sex man
*/
```



# 2. 临时增加struct字段

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Fixed struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	outputJson(Fixed{Name: "levon", Age: 9})
}

func outputJson(res interface{}) {
	data, _ := json.Marshal(res)
	fmt.Println(string(data))
}

// {"name":"levon","age":9}
```

Fixed假设是别人定义的结构体, 不能修改, 但是我要想让结构体增加一下当前的时间



### 2.1 临时struct增加字段

```go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

type Fixed struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	o := struct {
		Fixed
		Time string `json:"time"`
	}{
		Fixed: Fixed{Name: "levon", Age: 9},
		Time:  time.Now().Format("2006-01-02"),
	}
	outputJson(o)
}


func outputJson(res interface{}) {
	data, _ := json.Marshal(res)
	fmt.Println(string(data))
}


// {"name":"levon","age":9,"time":"2021-06-29"}
```



### 2.2 临时struct重写json

```go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

type Fixed struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

type NewFixed struct {
	Fixed
}

func (u *NewFixed) MarshalJSON() ([]byte, error) {
	type Alias NewFixed
	return json.Marshal(&struct {
		*Alias
		Time string `json:"time"`
	}{
		Alias: (*Alias)(u),
		Time:  time.Now().Format("2006-01-02"),
	})
}

func main() {
	outputJson(&NewFixed{Fixed{Name: "levonfly", Age: 9}})
}

func outputJson(res interface{}) {
	data, _ := json.Marshal(res)
	fmt.Println(string(data))
}


// {"name":"levon","age":9,"time":"2021-06-29"}
```



# 3. 临时为struct增加一个动态结构体
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Info struct {
	A string `json:"a"`
	B int    `json:"b"`
	C string `json:"c"`
}

func main() {
	info := &Info{A: "str1", B: 100, C: "str2"}
	outputJson(info)
}

func outputJson(res interface{}) {
	data, _ := json.Marshal(res)
	fmt.Println(string(data))
}

//{"a":"str1","b":100,"c":"str2"}
```

这时候加入一个动态的结构体, 例如下面有可能是1个d的结构, 也有可能是 2个属性e,f 的结构,  并把他们组合

```bash
{"d":"123"}
{"e":"123", "f":200}
```



解决思路就是倒入到一个 map[string]interface{} 里

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Info struct {
	A string `json:"a"`
	B int    `json:"b"`
	C string `json:"c"`
}

func main() {
	info := &Info{A: "str1", B: 100, C: "str2"}

	// 这里来了两个测试的动态结构
	vars1 := `{"d":"123"}`
	vars2 := `{"e":"123", "f":200}`

	outPuts := make(map[string]interface{})
	respByte, _ := json.Marshal(info)
	_ = json.Unmarshal(respByte, &outPuts)
	_ = json.Unmarshal([]byte(vars1), &outPuts)
	_ = json.Unmarshal([]byte(vars2), &outPuts)

	outputJson(outPuts)
}

func outputJson(res interface{}) {
	data, _ := json.Marshal(res)
	fmt.Println(string(data))
}

//{"a":"str1","b":100,"c":"str2","d":"123","e":"123","f":200}
```



# 4. 参考资料

+ https://zhuanlan.zhihu.com/p/27472716
+ http://choly.ca/post/go-json-marshalling/