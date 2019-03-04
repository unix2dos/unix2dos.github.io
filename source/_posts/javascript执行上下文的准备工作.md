---
title: "javascript执行上下文的准备工作"
date: 2019-03-07 22:52:01
tags:
- javascript
---



### 全局上下文的准备工作:

全局环境下 javascript真正运行语句之前, 解释器会做一些准备工作:

- 对变量的声明 (而变量的赋值, 是真正运行到那一行的时候才进行的.)
- 对全局变量 this 的赋值
- 对函数声明赋值, 对函数表达式声明



如何理解这三种情况呢?

1. 对变量的声明:

```javascript
console.log(a); // undefined
var a = 10;
```

准备工作是提前声明了变量 a, 和下面写法意思一样

```javascript
var a; 
console.log(a); // undefined
a = 10;
```

<!-- more -->

2. 对全局变量 this 的赋值:

```javascript
console.log(this) // window
```

执行前的准备工作是:  this 赋值为全局的 window 对象



3. 对函数声明赋值, 对函数表达式声明:

```javascript
console.log(f1);// function f1() {}
function f1() {} //这个是函数声明

console.log(f2);  // undefined
var f2 = function(){}; //这个是函数表达式
```

执行前的准备工作是:  函数声明 f1 被赋值, 函数表达式 f2 被声明



### 函数上下文额外的准备工作:

1. 函数定义时, 额外的准备工作:

- 记录函数内自由变量的作用域.  (自由变量就是引用外部作用域的变量)

```javascript
var a = 10;
function fn (){
    console.log(a); // a是自由变量,此处定义时就记录了a要取值的作用域,即全局作用域
}

function bar(f){
    var a = 20;
    f(); // 输出10, 而不是20, 因为fn函数内自由变量 a 的作用域早被记录下来了,是外面的10
}
bar(fn)
```



2. 函数调用时, 额外的准备工作:

- arguments 变量的赋值
- 函数参数的赋值

```javascript
function fn(x){
    console.log(arguments); // [10]
    console.log(x); // 10
}
fn(10);
```



### 来看一道题:

```javascript
var a = 'global';
var f = function(){
    console.log(a); // 答案是undefined, 想想为什么
    var a = 'local';
}
f();
```

