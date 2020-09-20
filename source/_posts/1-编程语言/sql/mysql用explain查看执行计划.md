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
  EXPLAIN SELECT name FROM  user_info \G
  ```

+ range

  表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.
  当 `type` 是 `range` 时, 那么 EXPLAIN 输出的 `ref` 字段为 NULL, 并且 `key_len` 字段是此次查询中使用到的索引的最长的那个.

+ index_merge

+ ref

  此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 `最左前缀` 规则索引的查询.

  ```sql
  EXPLAIN SELECT * FROM user_info, order_info WHERE user_info.id = order_info.user_id AND order_info.user_id = 5\G
  ```

+ eq_ref

  此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 `=`, 查询效率较高. 例如:

  ```sql
  EXPLAIN SELECT * FROM user_info, order_info WHERE user_info.id = order_info.user_id\G
  ```

+ const 

  针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.

  例如下面的这个查询, 它使用了主键索引, 因此 `type` 就是 `const` 类型的.

  ```sql
  explain select * from user_info where id = 2\G
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
  当 Extra 中有 `Using filesort` 时, 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果. 一般有 `Using filesort`, 都建议优化去掉, 因为这样的查询 CPU 资源消耗大.

+ Using index

  "覆盖索引扫描", 表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, 往往说明性能不错

+ Using temporary

  查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化.



### 1.5 构建数据

```sql
CREATE TABLE `user_info` (
  `id`   BIGINT(20)  NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL DEFAULT '',
  `age`  INT(11)              DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `name_index` (`name`)
)
  ENGINE = InnoDB
  DEFAULT CHARSET = utf8

INSERT INTO user_info (name, age) VALUES ('xys', 20);
INSERT INTO user_info (name, age) VALUES ('a', 21);
INSERT INTO user_info (name, age) VALUES ('b', 23);
INSERT INTO user_info (name, age) VALUES ('c', 50);
INSERT INTO user_info (name, age) VALUES ('d', 15);
INSERT INTO user_info (name, age) VALUES ('e', 20);
INSERT INTO user_info (name, age) VALUES ('f', 21);
INSERT INTO user_info (name, age) VALUES ('g', 23);
INSERT INTO user_info (name, age) VALUES ('h', 50);
INSERT INTO user_info (name, age) VALUES ('i', 15);


CREATE TABLE `order_info` (
  `id`           BIGINT(20)  NOT NULL AUTO_INCREMENT,
  `user_id`      BIGINT(20)           DEFAULT NULL,
  `product_name` VARCHAR(50) NOT NULL DEFAULT '',
  `productor`    VARCHAR(30)          DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_product_detail_index` (`user_id`, `product_name`, `productor`)
)
  ENGINE = InnoDB
  DEFAULT CHARSET = utf8

INSERT INTO order_info (user_id, product_name, productor) VALUES (1, 'p1', 'WHH');
INSERT INTO order_info (user_id, product_name, productor) VALUES (1, 'p2', 'WL');
INSERT INTO order_info (user_id, product_name, productor) VALUES (1, 'p1', 'DX');
INSERT INTO order_info (user_id, product_name, productor) VALUES (2, 'p1', 'WHH');
INSERT INTO order_info (user_id, product_name, productor) VALUES (2, 'p5', 'WL');
INSERT INTO order_info (user_id, product_name, productor) VALUES (3, 'p3', 'MA');
INSERT INTO order_info (user_id, product_name, productor) VALUES (4, 'p1', 'WHH');
INSERT INTO order_info (user_id, product_name, productor) VALUES (6, 'p1', 'WHH');
INSERT INTO order_info (user_id, product_name, productor) VALUES (9, 'p8', 'TE');
```





# 2. 测试

```bash
docker run -d --name mysql-explain -e MYSQL_ROOT_PASSWORD=123456 mysql # 创建一个容器

docker exec -it mysql-explain mysql -u root -p  # 输入密码, 进入操作
```

### 2.1 数据初始化

新建测试表，插入 10w 数据：

```sql
CREATE DATABASE test1;
use test1;

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


select count(*) from test;
-- 得到100000
```

### 2.2 全表查询

目前默认只有一个主键索引，我们分析下全表查询：

```
mysql> explain select * from test;  
```

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra | partitions | filtered |
| :--- | :---------- | :---- | :--- | :------------ | :--- | :------ | :--- | :----- | :---- | ---------- | -------- |
| 1    | SIMPLE      | test  | ALL  | NULL          | NULL | NULL    | NULL | 100098 | NULL  | NULL       | 100.00   |

其中 `type` 值为 ALL，表示全表扫描了，我们看到 `rows` 这个字段显示有 100098 条，实际上我们一共才 10w 条数据，说明这个字段只是 mysql 的一个预估，不总是准确的。

### 2.3 索引查询

接下来我们分别给字段 a 和 b 添加普通索引。

```
mysql> alter table test add index idx_a(a);  
mysql> alter table test add index idx_b(b);  
```

看下下面这条 sql：

```
mysql> explain select * from test where a > 10000;  
```

| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows   | Extra       | filtered | partitions |
| :--- | :---------- | :---- | :--- | :------------ | :--- | :------ | :--- | :----- | :---------- | -------- | ---------- |
| 1    | SIMPLE      | test  | ALL  | idx_a         | NULL | NULL    | NULL | 100098 | Using where | 50.00    | NULL       |

我们发现 `type` 竟然不是 index, 刚刚不是给字段 a 添加索引了么？还有 `possible_keys` 也显示了有 idx_a，但是 `key` 显示 null，表示实际上不会使用任何索引，这是为什么呢？

这是因为 select * 的话还需要回到主键索引上查找 b 字段，这个过程叫`回表`。

这条语句会从索引中查出 9w 条数据，也就是说这 9w 条数据都需要`回表`操作，全表扫描都才 10w 条数据，所以在 mysql 最后的决策是还不如直接全表扫描得了，至少还免去了回表过程了。



```
mysql> explain select a from test where a > 10000;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                         | filtered | partitions |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :---------------------------- | -------- | ---------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 50049 | Using where;<br/> Using index | 100.00   | NULL       |

注意这次 `Extra` 的值为 Using where; Using index，表示查询用到了索引，且要查询的字段在索引中就能拿到，所以不需要回表，显然这种效率比上面的要高，这也是日常开发中不建议写 select * 的原因，尽量只查询业务所需的字段。



当然，最后决策是否用索引不是固定的，mysql 会比较各种查询的代价，我们把上面的 sql 中 where 条件再稍微改造一下。

```
mysql> explain select * from test where a > 90000;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                 | filtered | partitions |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :-------------------- | -------- | ---------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using index condition | 100.00   | NULL       |

再看这次 `type` 为 range 了，`key` 为 a_index，表示使用了 a 索引，如我们所愿了。这是因为满足这次索引中查出只有 10000 条数据，mysql 认为 10000 条数据就算回表也要比全表扫描的代价低，因而决定查索引。

还有一点就是这次 `Extra` 字段中值为 Using index condition，这是指条件过滤的时候用到了索引，但因为是 select * ，所以还是需要回表。

上面两条查询说明 mysql 会比较 `索引 + 回表` 和 `直接全表扫描`的查询性能，选择其中更好的作为最后的查询方式，这就是 mysql 优化器的作用了。

### 2.4 排序查询

再来看一个带排序的查询。

```
mysql> explain select a from test where a > 90000 order by b;  
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows  | Extra                                      | filtered | partitions |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :---- | :----------------------------------------- | -------- | ---------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 10000 | Using index condition;<br/> Using filesort | 100.00   | NULL       |

我们知道索引本来就是有序带，但这个 `Extra` 中返回了一个 Using filesort，说明无法利用索引完成排序，需要从内存或磁盘进行排序，具体哪种排序 explain 是没有体现的。 

总之，这种情况也是需要优化的，尽量能利用索引的有序性，比如下面：

```
mysql> explain select a from test where a > 90000 order by a;
```

| id   | select_type | table | type  | possible_keys | key   | key_len | ref  | rows | Extra                         | filtered | partitions |
| :--- | :---------- | :---- | :---- | :------------ | :---- | :------ | :--- | :--- | :---------------------------- | -------- | ---------- |
| 1    | SIMPLE      | test  | range | idx_a         | idx_a | 4       | NULL | 9999 | Using where;<br/> Using index | 100.00   | NULL       |

这次 `Extra` 值有 Using index 了，表示使用上了索引。

### 2.5 复合索引

我们再创建一个复合索引看看。

```
mysql> alter table test drop index idx_a;
mysql> alter table test drop index idx_b;
mysql> alter table test add index idx_a_b(a,b);  
```

看下之前的查询 

```
mysql> explain select * from test where a > 10000;
```

| id   | select_type | table | type  | possible_keys | key     | key_len | ref  | rows  | Extra                         | filtered | partitions |
| :--- | :---------- | :---- | :---- | :------------ | :------ | :------ | :--- | :---- | :---------------------------- | -------- | ---------- |
| 1    | SIMPLE      | test  | range | idx_a_b       | idx_a_b | 4       | NULL | 50049 | Using where;<br/> Using index | 100.00   | NULL       |

这条 sql 刚刚在没有创建复合索引的时候，是走的全表扫描，现在看 `Extra` 有 Using index，说明利用了覆盖索引，同样也免去了回表过程，即在 idx_a_b 索引上就能找出要查询的字段。

# 3. 查看 sql 执行时间

```bash
set profiling = 1;
show profiles;
```



# 4. 参考资料

+ https://blog.souche.com/mysql-explain/
+ https://segmentfault.com/a/1190000008131735