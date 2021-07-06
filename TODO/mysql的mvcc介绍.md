---
title: "mysql的mvcc介绍"
date: 2021-07-06 00:00:00
tags:
- mysql
---

# 1. MVCC概念

多版本控制（Multiversion Concurrency Control）: 指的是一种提高并发的技术。最早的数据库系统，只有读读之间可以并发，读写，写读，写写都要阻塞。

引入MVCC之后，只有写写之间相互阻塞，其他三种操作都可以并行，这样大幅度提高了InnoDB的并发度。在内部实现中，InnoDB通过undo log保存每条数据的多个版本，并且能够找回数据历史版本提供给用户读，每个事务读到的数据版本可能是不一样的。在同一个事务中，用户只能看到该事务创建快照之前已经提交的修改和该事务本身做的修改。

**MVCC在 Read Committed 和 Repeatable Read两个隔离级别下工作。**

MySQL的InnoDB存储引擎默认事务隔离级别是RR(可重复读)，是通过 "行级锁+MVCC"一起实现的，正常读的时候不加锁，写的时候加锁。而 MVCC 的实现依赖：隐藏字段、Read View、Undo log。

<!-- more -->

### 1.1 隐藏字段

InnoDB存储引擎在每行数据的后面添加了三个隐藏字段：

+ DB_TRX_ID(6字节)：表示最近一次对本记录行作修改（insert | update）的事务ID。至于delete操作，InnoDB认为是一个update操作，不过会更新一个另外的删除位，将行表示为deleted。并非真正删除。

+ DB_ROLL_PTR(7字节)：回滚指针，指向当前记录行的undo log信息

+ DB_ROW_ID(6字节)：随着新行插入而单调递增的行ID。理解：当表没有主键或唯一非空索引时，innodb就会使用这个行ID自动产生聚簇索引。如果表有主键或唯一非空索引，聚簇索引就不会包含这个行ID了。(这个DB_ROW_ID跟MVCC关系不大)。

注意隐藏字段并没有什么创建版本、删除版本(网上流传)。



### 1.2 Read View

Read View（读视图），主要是用来做可见性判断的, 里面保存了“**对本事务不可见的其他活跃事务**”。(select 语句时候产生)

其中包括几个变量

+  low_limit_id：目前出现过的最大的事务ID+1，即下一个将被分配的事务ID。
+ up_limit_id：活跃事务列表trx_ids中最小的事务ID，如果trx_ids为空，则up_limit_id 为 low_limit_id。
+ trx_ids：Read View创建时其他未提交的活跃事务ID列表。意思就是创建Read View时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。
+ creator_trx_id：当前创建事务的ID，是一个递增的编号



### 1.3 Undo log    

Undo log中存储的是老版本数据，当一个事务需要读取记录行时，如果当前记录行不可见，可以顺着undo log链找到满足其可见性条件的记录行版本。

大多数对数据的变更操作包括 insert/update/delete，在InnoDB里，undo log分为如下两类：

+ insert undo log : 事务对insert新记录时产生的undo log, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
+ update undo log : 事务对记录进行delete和update操作时产生的undo log，不仅在事务回滚时需要，快照读也需要，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除。

##### 1.3.1 Purge线程

为了实现InnoDB的MVCC机制，更新或者删除操作都只是设置一下旧记录的deleted_bit，并不真正将旧记录删除。
为了节省磁盘空间，InnoDB有专门的purge线程来清理deleted_bit为true的记录。purge线程自己也维护了一个read view，如果某个记录的deleted_bit为true，并且DB_TRX_ID相对于purge线程的read view可见，那么这条记录一定是可以被安全清除的。

##### 1.3.2 其他log-binlog

是mysql服务层产生的日志，常用来进行数据恢复、数据库复制，常见的mysql主从架构，就是采用slave同步master的binlog实现的, 另外通过解析binlog能够实现mysql到其他数据源（如ElasticSearch)的数据复制。

##### 1.3.3 其他log-redolog

记录了数据操作在物理层面的修改，mysql中使用了大量缓存，缓存存在于内存中，修改操作时会直接修改内存，而不是立刻修改磁盘，当内存和磁盘的数据不一致时，称内存中的数据为脏页(dirty page)。

为了保证数据的安全性，事务进行中时会不断的产生redo log，在事务提交时进行一次flush操作，保存到磁盘中, redo log是按照顺序写入的，磁盘的顺序读写的速度远大于随机读写。

当数据库或主机失效重启时，会根据redo log进行数据的恢复，如果redo log中有事务提交，则进行事务提交修改数据。这样实现了事务的原子性、一致性和持久性。



# 2. MVCC流程








1.查看数据的事务隔离级别

SELECT @@tx_isolation;













+ https://zhuanlan.zhihu.com/p/66791480
+ https://zhuanlan.zhihu.com/p/52977862
+ https://blog.csdn.net/Waves___/article/details/105295060 (重点)
