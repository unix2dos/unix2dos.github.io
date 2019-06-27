---
title: javascript原形链的自我理解
tags:
  - javascript
abbrlink: 6d00c950
date: 2019-02-21 21:05:22
---

### Object 和 Function

首先在 javascript 中 我们要明确Object 和 Function两个概念:

+ 万物皆对象

+ 所有对象都是函数创建出来

仔细琢磨这两句话, 其实说的 Object 和 Function 是一个鸡生蛋还是蛋生鸡的问题. 为什么这么讲呢? 因为函数也是一个对象, 但对象又是函数创建出来的. 

 其实原形链的一切江湖恩怨都是围绕着Function和Object两大家族展开的.



<!-- more -->

### 鸡家族和蛋家族

+ 有创造力的函数我们称为Function鸡家族, 没有创造力的对象我们称为Object蛋家族

+ 芸芸众生,你我她都是一个蛋对象(~~). 我们都是被某一个函数创建出来, 我们可以称为创建我们的这个函数为鸡爸爸, 我们每人都有一个鸡爸爸

+ 注意创建我们的鸡爸爸函数, 也是一个对象. 它曾经也是被某一个函数创建出来的



### prototype鸡技能仓库

+ prototype 是 Function鸡家族独有的技能仓库, 没有创造能力的蛋家族没有这个仓库.

+ 鸡技能仓库内放着鸡爸爸的属性和函数等技能. 例如吃小米、捉虫子、变凤凰...

+ 注意鸡技能仓库属于 Object蛋家族,  因为鸡技能仓库没有创造能力, 鸡才有创造力



OK, 创建我们的鸡爸爸它拥有 prototype 属性.  意味着每个鸡都拥有一个技能仓库, 仓库的大门上也写着拥有者名字(constructor属性), 即这个鸡本人的名字

+ prototype 是鸡的属性, 而constructor是鸡技能仓库的属性

  

###  \__proto\__ 拥有鸡爸爸技能仓库的钥匙



+ \__proto\__ 是一个隐藏的指针, 万物皆有这个指针. 我们可以将这个 \__proto\__ 理解为 我拿着一把我鸡爸爸技能仓库的钥匙

+ 我拿了一把鸡爸爸技能仓库钥匙, 意味着我可以使用鸡爸爸技能仓库的技能,  例如吃小米、捉虫子、变凤凰...



那么鸡的 \__proto\__ 指向谁? 鸡技能仓库的  \__proto\__ 又指向谁?

+ 创建鸡(函数家族)的鸡爸爸 是function Function(),  所以鸡的  \__proto\__ 指向 Function.prototype

+ 创建鸡仓库(对象家族)的鸡爸爸是function  Object(), 所以鸡仓库的  \__proto\__ 指向 Object.prototype

+ 最后 Object. \__proto\__指向了 null



OK, 如果我调用一个函数, 那么先在我身上找这个函数. 

如果没找到, 就拿起我的钥匙去我鸡爸爸的技能仓库去找.

如果还没找到, 就拿起技能仓库对象的钥匙, 去仓库对象鸡爸爸的技能仓库去找, 一直找到 null为止.



### 分析一张图

![img](javascript原形链的自我理解/1.png)



上面部分:

+ f1 = new Foo()  我是f1, 没有prototype仓库,  我的  \__proto\__ 指向了鸡爸爸的仓库 Foo.prototype
+ 鸡爸爸(Function Foo)的技能仓库(prototype)是 Foo.prototype, 仓库的名字(constructor)是 function Foo()

+ 鸡技能仓库Foo.prototype的 \__proto\__ 指向了 Object.prototype,  Object.prototype的  \__proto\__ 指向了 null (前面已经提过)

左侧部分:

+ function Foo的鸡爸爸是 function Function(), 因为是这个函数创建了鸡爸爸, 所以 Function Foo的 \__proto\__指向了 Function.prototype 仓库
+ 再看function Function()  它的技能仓库是 Function.prototype, 技能仓库名字是 function Function() .  另外过分的是function Function() 是被它自己function Function()创建出来的, 所以它的 \__proto\__指向了自己的技能仓库

中间部分: 
+ function Object的  \__proto\__  指向了Function.prototype 看来所有鸡的钥匙都指向了Function.prototype
+ 技能仓库Function.prototype的 \__proto\__ 指向了 Object.prototype,  Object.prototype的  \__proto\__ 指向了 null 



### instanceof 寻仓库运算符

instanceof运算符的第一个变量是一个对象(蛋家族)，暂时称为A；第二个变量一般是一个函数(鸡家族)，暂时称为B。

Instanceof的判断队则是：沿着A的 \__proto\__ 这条线来找，同时沿着B的prototype这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回true。如果找到终点还未重合，则返回false。



```javascript
console.log(Object instanceof Function); //true
console.log(Function instanceof Object); //true
console.log(Function instanceof Function); //true
```



