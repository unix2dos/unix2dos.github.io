---
title: "javascript的this究竟指什么"
date: 2019-03-07 22:22:22
tags:
- javascript
---



在学习 javascript 的 this之前, 我们需要先明确一个重要知识:

> 在函数中this到底取何值，是在函数真正被调用执行的时候确定的，函数定义的时候确定不了。
>
> 简单记忆是，谁调用的函数，this就指向谁.



在 javascript 中, this的取值，一般分为下面几种情况。

### 一.  构造函数

+ new 出的对象:

```javascript
function Foo(){
  this.name = "levon";
  this.age = 26;
  console.log(this); // 当前构造的对象
}

var f1 = new Foo();
```



+ 直接运行函数(普通函数使用):

```javascript
function Foo(){
  this.name = "levon";
  this.age = 26;
  console.log(this); // 直接调用注意此处是 window
}

Foo();// window.Foo();
```

<!-- more -->

### 二.  函数作为对象的属性

+ 对象调用本身的函数:

```javascript
var obj = {
    x : 10,
    fn : function(){
        console.log(this);  // 此处就是 obj
        console.log(this.x);//10
    }
}

obj.fn();
```



+ 如果不用对象去调用:

```javascript
var obj = {
    x : 10,
    fn : function(){
        console.log(this);  // window
        console.log(this.x);// undefined
    }
}

var fn1 = obj.fn;
fn1();//window.fn1();

```



+ 需要注意的一种情况是下面的 f()函数, 此时 f()只是一个普通函数

```javascript
var obj = {
	x : 10,
	fn : function(){
		
        this;// 此处是 obj
        
		function f(){
			console.log(this); // window
			console.log(this.x);// undefined
		}
		f();
        
	}
}

obj.fn();
```

  

### 三: 函数用 call 或 apply 调用

+ 函数调用 call

```javascript
var obj = {
    x : 10
};

var fn = function(){
    console.log(this);  //函数调用call, this指向call中的参数, 就是obj
    console.log(this.x);// 10
}

fn.call(obj)
```



### 四:  全局环境下的 this

```javascript
console.log(this); // window


var x = 10; //--> window.x = 10
var fn = function(){
    console.log(this); //window
    console.log(this.x)//10
}
fn();// window.fn();
```



### 五: 原型链中的 this

```javascript
function Fn(){
    this.name = "levon";
}

Fn.prototype.getName = function(){
    console.log(this.name); //此处 this 是调用的当前对象 f1
}

var f1 = new Fn();
f1.getName(); //levon
```



其实不仅仅是构造函数的prototype，即便是在整个原型链中，this代表的也都是当前对象的值。