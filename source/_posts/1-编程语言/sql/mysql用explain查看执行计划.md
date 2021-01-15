---
title: mysql用explain查看执行计划
tags:
  - mysql
categories:
  - 1-编程语言
  - sql
abbrlink: 8a4c23bd
date: 2019-04-07 00:00:00
---

mysql 使用 `explain + sql 语句` 来查看执行计划，执行结果有以下字段，具体描述如下：

<!-- more -->


# 1. 字段说明

| 字段          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| id            | id相同，执行顺序由上至下；<br/>id不同，id的序号会递增，id值越大优先级越高，越先被执行<br/>id为`null`时表示一个结果集，不需要使用它查询，常出现在包含`union`等查询语句中。 |
| select_type   | 主要是用于区别普通查询、联合查询、子查询等的复杂查询         |
| table         | 当前执行的表                                                 |
| type          | 访问类型                                                     |
| possible_keys | 可能使用的索引                                               |
| key           | 实际使用的索引                                               |
| key_len       | 使用的索引的长度                                             |
| ref           | 显示索引的哪一列被使用了                                     |
| rows          | 查询过程中可能扫描的行数, 原则上 rows 越少越好.              |
| Extra         | 解析查询的额外信息，通常会显示是否使用了索引，是否需要排序，是否会用到临时表等 |
| partitions    | 匹配的分区                                                   |
| filtered      | 这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数。 |

### 1.1 select_type

| 类型               | 解释                                                         |
| ------------------ | ------------------------------------------------------------ |
| SIMPLE             | 不包含 UNION 查询或子查询                                    |
| PRIMARY            | 包含子查询最外层查询就显示为 `PRIMARY`                       |
| SUBQUERY           | 子查询中的第一个 SELECT                                      |
| DEPENDENT SUBQUERY | 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果. |
| UNION              | UNION 的第二或随后的查询                                     |
| DEPENDENT UNION    | UNION 中的第二个或后面的查询语句, 取决于外面的查询           |
| UNION RESULT       | UNION 的结果                                                 |




### 1.2 type

`type` 字段比较重要, 它提供了判断查询是否高效的重要依据依据. 通过 `type` 字段, 我们判断此次查询是 `全表扫描` 还是 `索引扫描` 等

从上到下性能从低到高排列:

+ all

  表示全表扫描, 这个类型的查询是性能最差的查询之一. 通常来说, 我们的查询不应该出现 ALL 类型的查询, 因为这样的查询在数据量大的情况下, 对数据库的性能是巨大的灾难. 如一个查询是 ALL 类型查询, 那么一般来说可以对相应的字段添加索引来避免.

+ index

  表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.

  `index` 类型通常出现在: 所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据. 当是这种情况时, Extra 字段 会显示 `Using index`.

  ```sql
  EXPLAIN SELECT name FROM  user_info
  ```

+ range

  表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.
  当 `type` 是 `range` 时, 那么 EXPLAIN 输出的 `ref` 字段为 NULL, 并且 `key_len` 字段是此次查询中使用到的索引的最长的那个.

+ index_merge

+ ref

  此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 `最左前缀` 规则索引的查询.

  ```sql
  EXPLAIN SELECT * FROM user_info, order_info WHERE user_info.id = order_info.user_id AND order_info.user_id = 5
  ```

+ eq_ref

  此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 `=`, 查询效率较高. 例如:

  ```sql
  EXPLAIN SELECT * FROM user_info, order_info WHERE user_info.id = order_info.user_id
  ```

+ const 

  针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.

  例如下面的这个查询, 它使用了主键索引, 因此 `type` 就是 `const` 类型的.

  ```sql
  explain select * from user_info where id = 2
  ```

+ system

  表中只有一条数据. 这个类型是特殊的 `const` 类型.
  
  

`ALL` 类型因为是全表扫描, 因此在相同的查询条件下, 它是速度最慢的.
而 `index` 类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比 ALL 类型的稍快.
后面的几种类型都是利用了索引来查询数据, 因此可以过滤部分或大部分数据, 因此查询效率就比较高了.



### 1.3 key_len

表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.
key_len 的计算规则如下:

- 字符串
  - char(n): n 字节长度
  - varchar(n): 如果是 utf8 编码, 则是 3 *n + 2字节; 如果是 utf8mb4 编码, 则是 4* n + 2 字节.
- 数值类型:
  - TINYINT: 1字节
  - SMALLINT: 2字节
  - MEDIUMINT: 3字节
  - INT: 4字节
  - BIGINT: 8字节
- 时间类型
  - DATE: 3字节
  - TIMESTAMP: 4字节
  - DATETIME: 8字节
- 字段属性: NULL 属性 占用一个字节. 如果一个字段是 NOT NULL 的, 则没有此属性.



### 1.4 Extra

+ Using filesort
  当 Extra 中有 Using filesort 时, 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果. 表明此时的查询无法利用索引完成排序操作即索引失效。

+ Using index

  "覆盖索引扫描", 表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, 往往说明性能不错

+ Using temporary

  查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化.



# 2. 测试

```bash
docker run -d --name mysql-explain -e MYSQL_ROOT_PASSWORD=123456 mysql # 创建一个容器

docker exec -it mysql-explain mysql -u root -p  # 输入密码, 进入操作
```



### 2.1 数据初始化

新建测试表，插入 10w 数据：

```sql
-- 创建库
CREATE DATABASE test1;
use test1;

-- 创建表
CREATE TABLE `test` (  
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `a` int(11) NOT NULL,
  `b` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

-- 批量插入 10w 数据
DROP PROCEDURE IF EXISTS batchInsert;
DELIMITER $  
CREATE PROCEDURE batchInsert() BEGIN DECLARE i INT DEFAULT 1;  
START TRANSACTION; WHILE i<=100000  
DO  
INSERT INTO test (a,b) VALUES (i,i);  
SET i=i+1; END WHILE;  
COMMIT; END $

CALL batchInsert();  


-- 得到100000
select count(*) from test;
```



### 2.2 没有索引

目前默认只有一个主键索引，我们分析下全表查询：

```sql
explain select * from test;  
```

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra | partitions | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :--- | :------ | :--- | :----- | :---- | ---------- | -------- |
| 1    | SIMPLE      | test  | ALL  | NULL          | NULL | NULL    | NULL | 100098 | NULL  | NULL       | 100.00   |

其中 `type` 值为 ALL，表示全表扫描了，我们看到 `rows` 这个字段显示有 100098 条，实际上我们一共才 10w 条数据，说明这个字段只是 mysql 的一个预估，不总是准确的。



### 2.3 单个字段索引

我们给字段 a 添加普通索引。

```sql
alter table test add index idx_a(a);  
```



##### 2.3.1 where 索引

+ 走 a 索引树,  虽1行也要回表。

```sql
explain select * from test where a = 10000;     
```

| id   | select_type | table | type | possible_keys | key   | key_len | ref   | rows | Extra | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :---- | :------ | :---- | :--- | :---- | -------- |
| 1    | SIMPLE      | test  | ref  | idx_a         | idx_a | 4       | const | 1    | NULL  | 100      |



+ 不走索引, 因为这条语句会从索引中查出 9w 条数据，全表扫描都才 10w 条数据，所以 mysql 决策是还不如直接全表扫描得了。

```sql
explain select * from test where a > 10000;  
```

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra       | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :--- | :------ | :--- | :----- | :---------- | -------- |
| 1    | SIMPLE      | test  | ALL  | idx_a         | NULL | NULL    | NULL | 100098 | Using where | 50.00    |



+ 使用了 a 索引树, 因为满足索引只有 10000 条数据，mysql 认为 10000 条数据就算回表也要比全表扫描的代价低，因而决定查索引，但还是需要回表。

```sql
explain select * from test where a > 90000;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                 | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :-------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using index condition | 100.00   |



+ 只select 索引字段, 注意这次 Extra 的值为 `Using where; Using index`，表示查询用到了索引，且要查询的字段在索引中就能拿到，所以不需要回表。显然这种效率比上面的要高，这也是日常开发中不建议写 select * 的原因，尽量只查询业务所需的字段。

```sql
explain select a from test where a > 10000;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 50049 | Using where; Using index | 100.00   |



##### 2.3.2 order by 索引

order by 最好和 where 用同一个字段



+ 走 a 索引树, 不回表

```sql
explain select a from test where a > 90000 order by a;
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using where; Using index | 100.00   |



+ 走 a 索引树, 需要回表

```sql
explain select * from test where a > 90000 order by a;
explain select b from test where a > 90000 order by a;
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                 | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :-------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using index condition | 100.00   |



+ Extra中返回了一个 Using filesort，说明无法利用索引完成排序，需要从内存或磁盘进行排序。就算 b 有索引, 也无法避免, 因为也会走 a 的索引树。

``` sql
explain select a from test where a > 90000 order by b;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                                 | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :------------------------------------ | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using index condition; Using filesort | 100.00   |



### 2.4 多个字段索引和复合索引

##### 2.4.1 多个字段索引

目前a, b 有自己的单个索引。

```sql
alter table test drop index idx_a_b;
alter table test add index idx_a(a);  
alter table test add index idx_b(b);
```


+ 没走索引,是因为无论如何都需要回表, 如果把10000改成90000, 会变成走索引加回表.

```sql
explain select * from test where a > 10000;
explain select a,b from test where a > 10000;
explain select b from test where a > 10000;
```

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra       | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :--- | :------ | :--- | :----- | :---------- | -------- |
| 1    | SIMPLE      | test  | ALL  | idx_a         | NULL | NULL    | NULL | 100098 | Using where | 50       |



+ 直接走a 的索引树, 不回表

```sql
explain select a from test where a > 10000;
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 50049 | Using where; Using index | 100      |



+ 走 b 的索引树更好,回表

```sql
explain select a,b from test where a > 90000 and b = 90000;
```

| id   | select_type | table | type | possible_keys | key   | key_len | ref   | rows | Extra       | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :---- | :------ | :---- | :--- | :---------- | -------- |
| 1    | SIMPLE      | test  | ref  | idx_a,idx_b   | idx_b | 4       | const | 1    | Using where | 9.99     |



+ 走 a 的索引树更好,回表

```sql
explain select a,b from test where a = 90000 and b > 90000;
```

| id   | select_type | table | type | possible_keys | key   | key_len | ref   | rows | Extra       | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :---- | :------ | :---- | :--- | :---------- | -------- |
| 1    | SIMPLE      | test  | ref  | idx_a,idx_b   | idx_a | 4       | const | 1    | Using where | 9.99     |



##### 2.4.2 一个复合索引

```sql
alter table test drop index idx_a;
alter table test drop index idx_b;
alter table test add index idx_a_b(a,b);  
```

目前只有一个(a,b)复合索引



+ 走 (a,b) 的索引树, 不回表

```sql
explain select * from test where a > 10000;
explain select a,b from test where a > 10000;
explain select b from test where a > 10000;
explain select a from test where a > 10000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b       | idx_a_b | 4       | NULL | 50049 | Using where; Using index | 100      |



+ 不满足最左匹配原则

```sql
explain select * from test where b > 10000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows   | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :----- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | index | NULL          | idx_a_b | 8       | NULL | 100098 | Using where; Using index | 33.33    |



+ 走 (a,b) 的索引树, 不回表, key_len=4

```sql
explain select a,b from test where a > 90000 and b = 90000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b       | idx_a_b | 4       | NULL | 18142 | Using where; Using index | 10       |



+ 走 (a,b) 的索引树, 不回表, key_len=8

```sql
explain select a,b from test where a = 90000 and b > 90000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :--- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b       | idx_a_b | 8       | NULL | 1    | Using where; Using index | 100      |



##### 2.4.3 普通索引和复合索引同时存在

```sql
alter table test add index idx_a(a); 
alter table test add index idx_b(b);  
alter table test add index idx_a_b(a,b);  
```

目前有2个普通索引,1个复合索引. 



+ 走 (a,b) 的索引树更好, 不回表

```sql
explain select * from test where a > 10000;
explain select a,b from test where a > 10000;
explain select b from test where a > 10000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b,idx_a | idx_a_b | 4       | NULL | 50049 | Using where; Using index | 100      |



+ 走 a 的索引树更好, 不回表

```sql
explain select a from test where a > 10000;
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b,idx_a | idx_a | 4       | NULL | 50049 | Using where; Using index | 100      |



+ 走 b 的索引树,  需要回表. 其实此时走复合索引更好, 所以不建议复合索引和普通索引一起用

```sql
explain select a,b from test where a > 90000 and b = 90000;
```

| id   | select_type | table | type | possible_keys       | key   | key_len | ref   | rows | Extra       | filtered |
| :--- | :---------- | :---- | :--- | :------------------ | :---- | :------ | :---- | :--- | :---------- | -------- |
| 1    | SIMPLE      | test  | ref  | idx_a_b,idx_a,idx_b | idx_b | 4       | const | 1    | Using where | 18.12    |



+ 走 (a,b) 的索引树, 不回表

```sql
explain select a,b from test where a = 90000 and b > 90000;
```

| id   | select_type | table | type  | possible_keys       | key     | key_len | ref  | rows | Extra                    | filtered |
| :--- | :---------- | :---- | :---- | :------------------ | :------ | :------ | :--- | :--- | :----------------------- | -------- |
| 1    | SIMPLE      | test  | range | idx_a_b,idx_a,idx_b | idx_a_b | 8       | NULL | 1    | Using where; Using index | 100      |

##### 2.4.4  复合索引和单个索引占用空间

复合索引(a,b,c)和单列索引(a)在查询条件中只有a字段时都会被调用,但是存在较小的性能差异.主要性能差异取决于IO性能. 

因为索引实质上就是将该条记录的唯一标识rowid和索引字段记录下来.所以一个最小的存储物理块block存储单列索引(a)可能可以存储100条数据,而复合索引(a,b,c)可能只能存储30条,那么在使用索引时,复合索引需要IO到的block更多,自然就会存在性能差异.

但是在表空间的使用上,复合索引的占用空间并不等于所涉及到的列的所有单列索引的总和,而是远远小于,经实测4个字段的复合索引占用空间为3072KB,而4个单列索引占用的空间均为2048KB,即4个单列索引总和为8192KB.主要原因是每个索引中都必须包含rowid,而rowid是比较大的,通常情况下占用空间远大于数据.



# 3. 其他

### 3.1 查看 sql 执行时间

```bash
set profiling = 1;
show profiles;
```

### 3.2 索引长度

对于myisam和innodb存储引擎，prefixes的长度限制分别为1000 bytes和767 bytes。注意prefix的单位是bytes，但是建表时我们指定的长度单位是字符。 

以utf8字符集为例，一个字符占3个bytes。因此在utf8字符集下，对myisam和innodb存储引擎创建索引的单列长度不能超过333个字符和255个字符。 

smallint 占2个bytes，timestamp占4个bytes，utf8字符集。utf8字符集下，一个character占3个bytes。 

### 3.3 key_len

完全匹配, 就是 key_len 是满的, 不完全匹配, 例如前面, key_len 就是少的

例如: (user_id, award_id, lottery_id)  复合索引, 一共是12

```sql
explain SELECT * FROM `lottery_user_log` WHERE (user_id=279223)  --key_len:4

explain SELECT * FROM `lottery_user_log` WHERE (user_id=279223 and award_id = 8)  --key_len:8

explain SELECT * FROM `lottery_user_log` WHERE (user_id=279223 and award_id = 8 and lottery_id=1) --key_len:12
```



### 3.4 不满足最左匹配, 也用了索引

> 就是  possible_keys is null but key isn't null

key可能有一个不存在于possible_keys值中的索引。

如果所有可能的索引都不适合查找行，但查询选择的所有列都是其他索引的列，则会发生这种情况。

也就是说，命名索引覆盖选定的列尽管它不用于确定要检索的行，但索引扫描比数据行扫描更有效。

https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key



# 4. 头脑风暴

+ order by 最好和 where 用同一个字段, 能有效避免 Using filesort
+ 尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

# 5. 参考资料

+ https://blog.souche.com/mysql-explain/
+ https://segmentfault.com/a/1190000008131735