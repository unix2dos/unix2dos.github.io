---
title: mysql事务隔离级别和表锁行锁
tags:
  - mysql
categories:
  - 1-编程语言
  - sql
abbrlink: 9762ea3e
date: 2019-04-07 00:00:00
---

# 1. 事务

### 1.1 事务的四大特性

+ 原子性
+ 一致性
+ 隔离性
+ 持久性

mysql 默认是开启事务的, 因为有默认提交(autocommit=on).就是普通的 upddate 也是一个事务.

<!-- more -->

### 1.2 并发事务引起的问题

+ 脏读
另外一个事务没提交你就读到

+ 不可重复读
另外一个事务提交后你就读到,  update/delete

+ 幻读
新增了一个数据, 你读到了     insert

### 1.3 事务隔离级别

| 类型                         | 备注                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| Read Uncommitted(RU)未提交读 | 不加锁                                                       |
| Read Committed(RC)已提交读   | 解决脏读                                                     |
| Repeatable Read(RR)可重复读  | 解决脏读, 不可重复读, innodb 解决了幻读(因为间隙锁不让别人插入) |
| Serializable(SE) 串行化      | select隐式转为lock in share mode, 会和 update,delete 互斥  解决脏读, 不可重复读, 幻读 |

### 1.4 事务隔离解决方案

+ 读取数据前, 对其加锁, 阻止其他事务修改  LBCC(Lock Based Concurrency Control)
+ (常用)生成数据请求时间点的一致性数据快照, 用快照提供一定级别的一致性读取 MVCC(Multi version Concurrency Control)   



# 2. 锁

myisam 只支持表锁, innodb支持表锁和行锁.

### 2.0 预热

+ 普通的 select 语句(没有加锁后缀), innodb 不会加锁, 就算别人锁了, 你也可以读
+ lock  in share mode 读锁, 只能读, 其他人不能改, 自己也不能改,  别人可以再加个读锁, 不能再加写锁
+ 手动加锁的话, commit/rollback 释放锁
+ delete/update/insert 默认自动加写锁



### 2.1 锁的模式

| 锁                 | 类别 | 使用                                                      |
| ------------------ | ---- | --------------------------------------------------------- |
| 共享锁(读锁)(S 锁) | 行锁 | select * from table where id = 1<br/>lock  in share mode; |
| 排它锁(写锁)(X 锁) | 行锁 | select * from table where id = 1<br/>for update;          |
| 意向共享锁         | 表锁 | 数据引擎自己维护,用户无法手动操作                         |
| 意向排他锁         | 表锁 | 数据引擎自己维护,用户无法手动操作                         |

加表锁的前提:

+ 全表扫描, 确认表里没有行, 被其他事务加锁, 所以才能成功加表锁
+ 意向锁类似一个标志, 能提高加 表锁 的效率



### 2.2 行锁算法 

假设数据是 1, 4, 7, 10

| 算法                 | 描述                                            | 使用                                                         | 备注         |
| -------------------- | ----------------------------------------------- | :----------------------------------------------------------- | ------------ |
| Record Lock记录锁    | 唯一主键, 精准匹配                              | select * from t where id=4 for update; <br/>锁住 id=4        |              |
| Gap Lock间隙锁       | 完全避过了主键记录, 锁住数据不存在的区间        | select * from where id > 4 and id < 7 for update; <br/>锁住4-7的区间 | 为了不让插入 |
| Next-key Lock 临键锁 | = Record Lock + Gap Lock,包含记录和不存在的区间 | select * from where id > 5 and id < 9 for update;<br/>锁住4-7, 7-10的区间 |              |



### 2.3 锁到底锁住了什么

锁其实是锁住了索引

+ 如果不使用索引
走表锁


+ 使用了索引
走行锁, 锁住了是索引, 不是锁住了一行


+ 但是为什么2个索引, 锁了一个索引, 第二个索引也被锁了?

  因为索引分为两类: 

  聚集索引(主键/ Unique Not null/ _rowId 字段 )  存储索引和数据(B+树叶子节点)
  二级索引 存储索引和主键值

  所以对二级索引加锁, 也会导致对主键加锁, 因为二级索引要根据主键去 B+树 叶子节点找值



# 3. 参考资料

+ [MySQL事务和锁机制详解](https://www.bilibili.com/video/BV1x54y1979n?from=search&seid=4833652458207423339)

