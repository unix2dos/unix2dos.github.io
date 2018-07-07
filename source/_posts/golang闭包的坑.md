---
title: "golang闭包的坑"
date: 2018-07-07 13:33:01
tags:
- golang
---

### 循环内goroutine使用闭包

```
func main() {                
    s := []string{"a", "b", "c"}                             
    for _, v := range s { 
        go func() {
            fmt.Println(v)
        }()                 
    }                                                                             
}
```

改进:

```
func main() {                
    s := []string{"a", "b", "c"}                             
    for _, v := range s { 
        go func(v string) {
            fmt.Println(v)
        }(v)      
    }                                                                            
}
```

<!-- more -->

## 循环内闭包函数列表

```
func test() []func() {
    var s []func()

    for i := 0; i < 3; i++ {
        s = append(s, func() {  
            fmt.Println(&i, i)
        })
    }

    return s    
}
func main() {
    for _, f := range test() { 
        f()   
    }
}
```

改进:

```
func test() []func() {
    var s []func()
    
    for i := 0; i < 3; i++ {
        x := i                 
        s = append(s, func() {
            fmt.Println(&x, x)
        })
    }

    return s
}
func main() {
    for _, f := range test() {
        f()
    }
}
```



### defer延迟调用闭包

```
func main() {
    x, y := 1, 2

    defer func(a int) { 
        fmt.Printf("x:%d,y:%d\n", a, y)  // y 为闭包引用
    }(x) // 复制 x 的值

    x += 100
    y += 100
    fmt.Println(x, y)
}


101 102
x:1,y:102
```
