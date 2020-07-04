---
title: "redis学习"
date: 2019-04-07 00:00:00
tags:
- redis
---

# 0. 前言

Redis是一个开源的，**基于内存的数据结构存储**，可用作于数据库、**缓存**、消息中间件。

- 从官方的解释上，我们可以知道：Redis是基于内存，支持多种数据结构。
- 从经验的角度上，我们可以知道：Redis常用作于缓存。

<!-- more -->

# 1. Redis的数据结构

Redis支持丰富的数据结构，**常用**的有string、list、hash、set、sortset这几种。学习这些数据结构是使用Redis的基础！

首先还是得声明一下，Redis的存储是以`key-value`的形式的。Redis中的key一定是字符串，value可以是string、list、hash、set、sortset这几种常用的。



### 1.1 字符串(stirng)对象

在上面的图我们知道string类型有三种**编码格式**：

- int：整数值，这个整数值可以使用long类型来表示

- - 如果是浮点数，那就用embstr或者raw编码。具体用哪个就看这个数的长度了

- embstr：字符串值，这个字符串值的长度小于32字节

- raw：字符串值，这个字符串值的长度大于32字节

embstr和raw的**区别**：

- raw分配内存和释放内存的次数是两次，embstr是一次
- embstr编码的数据保存在一块**连续**的内存里面

编码之间的**转换**：

- int类型如果存的**不再是一个整数值**，则会从int转成raw
- embstr是只读的，在修改的时候回从embstr转成raw



















# 5. 参考资料

+ https://github.com/ZhongFuCheng3y/3y