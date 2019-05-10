---
title: "nodejs介绍和javascript的区别"
date: 2019-05-10 20:00:00
tags:
- node
- javascript
---

### 1. nodejs的诞生

话说有个叫Ryan Dahl的歪果仁，他的工作是用C/C++写高性能Web服务。对于高性能，异步IO、事件驱动是基本原则，但是用C/C++写就太痛苦了。于是这位仁兄开始设想用高级语言开发Web服务。他评估了很多种高级语言，发现很多语言虽然同时提供了同步IO和异步IO，但是开发人员一旦用了同步IO，他们就再也懒得写异步IO了，所以，最终，Ryan瞄向了JavaScript。

因为JavaScript是单线程执行，根本不能进行同步IO操作，所以，JavaScript的这一“缺陷”导致了它只能使用异步IO。

选定了开发语言，还要有运行时引擎。这位仁兄曾考虑过自己写一个，不过明智地放弃了，因为V8就是开源的JavaScript引擎。让Google投资去优化V8，咱只负责改造一下拿来用，还不用付钱，这个买卖很划算。

于是在2009年，Ryan正式推出了基于JavaScript语言和V8引擎的开源Web服务器项目，命名为Node.js。虽然名字很土，但是，Node第一次把JavaScript带入到后端服务器开发，加上世界上已经有无数的JavaScript开发人员，所以Node一下子就火了起来。

<!-- more -->

>  在Node上运行的JavaScript相比其他后端开发语言有何优势？

最大的优势是借助JavaScript天生的事件驱动机制加V8高性能引擎，使编写高性能Web服务轻而易举。

其次，JavaScript语言本身是完善的函数式语言，在前端开发时，开发人员往往写得比较随意，让人感觉JavaScript就是个“玩具语言”。但是，在Node环境下，通过模块化的JavaScript代码，加上函数式编程，并且无需考虑浏览器兼容性问题，直接使用最新的ECMAScript 6标准，可以完全满足工程上的需求。



### 2. nodeJs和javascript的异同

我相信很多入坑Nodejs的人都是前端转过来的，但是局限于公司项目用不到Nodejs，只能自学，有些重要且基础的东西就忽略了。

前端的JavaScript其实是由ECMAScript、DOM、BOM组合而成。JavaScript=ECMAScript+DOM+BOM。

#### 2.1 javascript：

- ECMAScript(语言基础，如：语法、数据类型结构以及一些内置对象)
- DOM（一些操作页面元素的方法）
- BOM（一些操作浏览器的方法）

上面是JavaScript的组成部分，那么Nodejs呢？

#### 2.2 nodejs：

- ECMAScript(语言基础，如：语法、数据类型结构以及一些内置对象)
- os(操作系统)
- file(文件系统)
- net(网络系统)
- database(数据库)

分析：很容易看出，前端和后端的js相同点就是，他们的语言基础都是ECMAScript，只是他们所扩展的东西不同，前端需要操作页面元素，于是扩展了DOM，也需要操作浏览器，于是就扩展了BOM。而服务端的js则也是基于ECMAScript扩展出了服务端所需要的一些API，稍微了解后台的童鞋肯定知道，后台语音有操作系统的能力，于是扩展os，需要有操作文件的能力，于是扩展出file文件系统、需要操作网络，于是扩展出net网络系统，需要操作数据，于是要扩展出database的能力。

这么一对比，相信很多小伙伴对nodejs更加了解了，原来前端和服务端的js如此相似，他们的基础是相同的，只是环境不同，导致他们扩展出来的东西不同而已。



### 3. 总结:

在ecmascript部分node和JS其实是一样的，比如与数据类型的定义、语法结构，内置对象。

例如js中的顶层对象是window对象，但是在node中没有什么window对象，node中的顶层对象是global对象。