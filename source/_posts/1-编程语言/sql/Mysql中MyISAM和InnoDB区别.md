---
title: Mysql中MyISAM和InnoDB区别
tags:
  - mysql
categories:
  - 1-编程语言
  - sql
abbrlink: f9fde0e9
date: 2019-03-09 00:00:00
---



# 1. MySQL存储引擎

### 1.1 默认存储引擎的变迁

在MySQL 5.1之前的版本中，默认的搜索引擎是MyISAM，从MySQL 5.5之后的版本中，默认的搜索引擎变更为InnoDB。

InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一。

InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一。

<!-- more -->

### 1.2 MyISAM

MyISAM存储引擎的特点是：表级锁、**不支持事务**和 全文索引，适合一些CMS内容管理系统作为后台数据库使用，但是使用大并发、重负荷生产系统上，表锁结构的特性就显得力不从心；

MyISAM适合：

（1）做很多count 的计算；

（2）插入不频繁，查询非常频繁，如果执行大量的SELECT，MyISAM是更好的选择；

（3）没有事务。

### 1.3 InnoDB

InnoDB存储引擎的特点是：行级锁、事务安全（ACID兼容）、支持外键、不支持FULLTEXT类型的索引(5.6.4以后版本开始支持FULLTEXT类型的索引)。

InnoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全存储引擎。InnoDB是为处理巨大量时拥有最大性能而设计的。它的CPU效率可能是任何其他基于磁盘的关系数据库引擎所不能匹敌的。

InnoDB适合：

（1）可靠性要求比较高，或者要求事务；

（2）表更新和查询都相当的频繁，并且表锁定的机会比较大的情况指定数据引擎的创建；

（3）如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表；

（4）DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的 删除；

（5）LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用。



要注意，创建每个表格的代码是相同的，除了最后的 TYPE参数，这一参数用来指定数据引擎。



# 2. 具体区别

+ 存储结构

MyISAM：每个MyISAM在磁盘上存储成三个文件。分别为：表定义文件、数据文件、索引文件。第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为.MYD (MYData)。索引文件的扩展名是.MYI (MYIndex)。

InnoDB：所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB表的大小只受限于操作系统文件的大小，一般为2GB。



+ 索引

MyISAM: 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

InnoDB: 是聚集索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。



+ 存储空间

MyISAM： MyISAM支持支持三种不同的存储格式：静态表(默认，但是注意数据末尾不能有空格，会被去掉)、动态表、压缩表。当表在创建之后并导入数据之后，不会再进行修改操作，可以使用压缩表，极大的减少磁盘的空间占用。

InnoDB： 需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。



+ 可移植性、备份及恢复

MyISAM：数据是以文件的形式存储，所以在跨平台的数据转移中会很方便。在备份和恢复时可单独针对某个表进行操作。

InnoDB：免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了。



+ 事务支持

MyISAM：强调的是性能，每次查询具有原子性,其执行数度比InnoDB类型更快，但是不提供事务支持。

InnoDB：提供事务支持事务，外部键等高级数据库功能。 具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。



+ AUTO_INCREMENT 

MyISAM：可以和其他字段一起建立联合索引。引擎的自动增长列必须是索引，如果是组合索引，自动增长可以不是第一列，他可以根据前面几列进行排序后递增。

InnoDB：InnoDB中必须包含只有该字段的索引。引擎的自动增长列必须是索引，如果是组合索引也必须是组合索引的第一列。



+ 表锁差异

MyISAM： 只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。

InnoDB： 支持事务和行级锁，是innodb的最大特色。行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的。



+ 全文索引

MyISAM：支持 FULLTEXT类型的全文索引

InnoDB：不支持FULLTEXT类型的全文索引，但是innodb可以使用sphinx插件支持全文索引，并且效果更好。



+ 表主键

MyISAM：允许没有任何索引和主键的表存在，索引都是保存行的地址。

InnoDB：如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)，数据是主索引的一部分，附加索引保存的是主索引的值。



+ 表的具体行数

MyISAM： 保存有表的总行数，如果select count(*) from table;会直接取出出该值。*

InnoDB： 没有保存表的总行数，如果使用select count(*) from table；就会遍历整个表，消耗相当大，但是在加了wehre条件后，myisam和innodb处理的方式都一样。



+ CRUD操作

MyISAM：如果执行大量的SELECT，MyISAM是更好的选择。

InnoDB：如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表。



+ 外键

MyISAM：不支持

InnoDB：支持



# 3. 参考资料

+ [MySQL存储引擎－－MyISAM与InnoDB区别](https://link.zhihu.com/?target=https%3A//segmentfault.com/a/1190000008227211)

