---
title: "sql查询常见知识"
date: 2021-01-12 00:00:00
tags:
- sql
---

# 1. 关键词

### 1.1 union 和  union all 区别

UNION removes duplicate records (where all columns in the results are the same)

UNION ALL does not.

<!-- more -->

### 1.2 distinct的用法

语法:

```sql
SELECT DISTINCT column_name,column_name FROM table_name; -- 对后面所有的列, 共同起效果
```

**1.2.1 group by 和 distinct 的区别**

GROUP BY lets you use aggregate functions, like AVG, MAX, MIN, SUM, and COUNT. 

On the other hand DISTINCT just removes duplicates.

如果想达到同样效果, 可以用distinct, 因为更快一些.



# 2. NULL

### 2.1 is null 和  is not null

语法:

```sql
SELECT column_names FROM table_name WHERE column_name IS NULL;
SELECT column_names FROM table_name WHERE column_name IS NOT NULL;
```



判断一个字符串是否为空字符串?

```sql
Select * From Table Where (col is null or col = '')
```



### 2.2 not in 和 null 的结合

> SQL uses three valued logic: true, false, and unknown. A comparison with null results in unknown, which is not true

创建表:

```sql
CREATE TABLE pony
(
id INT PRIMARY KEY,
name VARCHAR(255)
);

INSERT INTO pony (id, name)
VALUES
(1, ‘Twilight Sparkle’),
(2, ‘Rainbow Dash’),
(3, ‘Pinkie Pie’),
(4, ‘Rarity’),
(5, ‘Applejack’);
```



**注意 NULL 不能直接 !=** 

```sql
SELECT * FROM pony;
— 5 rows as expected

SELECT * FROM pony WHERE id = NULL;
— 0 rows as expected

SELECT * FROM pony WHERE id != NULL;
— 0 rows, slight wtf

SELECT * FROM pony WHERE id IS NOT NULL;
— 5 rows as expected
```



**NULL still works intuitively when using WHERE IN:**

```sql
SELECT * FROM pony WHERE id IN (1, 2, NULL);
— 2 rows as expected

— equivalent statement:

SELECT * FROM pony
WHERE id = 1
OR id = 2
OR id = null;
```



**WHERE NOT IN is where things get tricky:**

```sql
SELECT * FROM pony
WHERE id NOT IN (1, 2, NULL);
— 0 rows, major wtf


SELECT * FROM pony
WHERE id != 1
AND id != 2
AND id != NULL;
```

Like explained in the intro, id != NULL is always unknown, therefor the entire WHERE clause is always FALSE.





# 3. 子查询

SQL子查询可以分为相关子查询和非相关子查询两类。

### 3.1 非相关子查询

非相关子查询的执行不依赖与外部的查询。非相关子查询一般可以分为：返回单值的子查询和返回一个列表的子查询。

执行过程：
（1）执行子查询，其结果不被显示，而是传递给外部查询，作为外部查询的条件使用。
（2）执行外部查询，并显示整个结果。　　

**3.1.1 返回单值：**
查询所有价格高于平均价格的图书名，作者，出版社和价格。

```sql
SElECT 图书名，作者，出版社，价格
  FROM Books
  WHERE 价格 >
  (
    SELECT AVG(价格)
    FROM Books
  )
```

**3.1.2 返回值列表:**

```sql
查询所有借阅图书的读者信息
SElECT *
  FROM Readers
  WHERE 读者编号 IN
  (
    SELECT 读者编号
    FROM [Borrow History]
  )
```



### 3.2 相关子查询

相关子查询的执行依赖于外部查询, 相关子查询无法独立于外部查询而得到解决。

执行过程：
（1）从外层查询中取出一个元组，将元组相关列的值传给内层查询。
（2）执行内层查询，得到子查询操作的值。
（3）外查询根据子查询返回的结果或结果集得到满足条件的行。
（4）然后外层查询取出下一个元组重复做步骤1-3，直到外层的元组全部处理完毕。 　　



查询Books表中大于该类图书价格平均值的图书信息

```sql
SELECT 图书名，作者，出版社，价格 FROM Books As a
  WHERE 价格 >
  (
    SELECT AVG(价格)
    FROM Books AS b
    WHERE a.类编号=b.类编号
  )
```

### 3.3 总结

非相关子查询是独立于外部查询的子查询，子查询总共执行一次，执行完毕后将值传递给外部查询。
相关子查询的执行依赖于外部查询的数据，外部查询执行一行，子查询就执行一次。
故非相关子查询比相关子查询效率高。



# 4. in 和 exist

### 4.1 in

语法:

```sql
SELECT Column(s) FROM table_name WHERE column IN (value1, value2, ... valueN);
SELECT Column(s) FROM table_name WHERE column IN (SELECT Statement);
```


即可以用在子查询, 可以不用


### 4.2 exist

语法： EXISTS subquery
参数：  subquery 是一个受限的 SELECT 语句 (不允许有 COMPUTE 子句和 INTO 关键字)。
结果类型： Boolean 如果子查询包含行，则返回 TRUE ，否则返回 FLASE 。

```sql
SELECT 
c.CustomerId,c.CompanyName 
FROM 
Customers c
WHERE 
EXISTS(
SELECT o.OrderID FROM Orders o WHERE o.CustomerID=c.CustomerID
)
```

这里面的EXISTS是如何运作呢？子查询返回的是OrderId字段，可是外面的查询要找的是CustomerID和CompanyName字段，这两个字段肯定不在OrderID里面啊，这是如何匹配的呢？

EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False

 如果EXISTS子句返回TRUE，这一行可作为外查询的结果行，否则不能作为结果。



### 4.3 in vs exist

**4.3.1 in 解析**

```sql
select * from A  where id in(select id from B)
```

以上查询使用了in语句,in( )只执行一次**,它查出B表中的所有id字段并缓存起来**.

之后,检查A表的id是否与B表中的id相等,如果相等则将A表的记录加入结果集中,直到遍历完A表的所有记录. 它的查询过程类似于以下过程:

```java
List resultSet=[];
Array A=(select * from A);
Array B=(select id from B);
for(int i=0;i<A.length;i++) {
   for(int j=0;j<B.length;j++) {
      if(A[i].id==B[j].id) {
         resultSet.add(A[i]);
         break;
      }
   }
}
return resultSet;
```

可以看出,当B表数据较大时不适合使用in( ),因为它会B表数据全部遍历一次.

**4.3.2 exist 解析**

```sql
select a.* from A a  where exists(select 1 from B b where a.id=b.id)
```

以上查询使用了exists语句,exists( )会执行A.length次,它并不缓存exists( )结果集

因为exists( )结果集的内容并不重要,重要的是结果集中是否有记录,如果有则返回true,没有则返回false. 它的查询过程类似于以下过程

```java
List resultSet=[];
Array A=(select * from A)
for(int i=0;i<A.length;i++) {
   if(exists(A[i].id) {    //执行select 1 from B b where b.id=a.id是否有记录返回
       resultSet.add(A[i]);
   }
}
return resultSet;
```


当B表比A表数据大时适合使用exists( ),因为它没有那么遍历操作,只需要再执行一次查询就行.

### 4.4 总结

+  In可以与子查询一起使用,也可以直接in (a,b…..)。exist, not exist一般都是与子查询一起使用.

+ 注意,一直以来认为exists比in效率高的说法是不准确的。

  ```
  in 与子查询一起使用的时候,只能针对主查询使用索引
  not in则不会使用任何索引.
  
  exist会针对子查询的表使用索引. 
  not exist会对主子查询都会使用索引.
  ```

+ IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。

+ 如果选择NOT IN  和 NOT EXISTS, 要注意 NOT IN 和 NULL 结合的问题。



# 5. 查询注意点

### 5.1 like 不要查前缀

``` sql
WHERE Field LIKE '%blah%'
```

That causes a table/index scan, because the LIKE value begins with a wildcard character.

### 5.2 字段不要用函数

```sql
WHERE FUNCTION(Field) = 'BLAH'
```

The database server will have to evaluate FUNCTION() against every row in the table and then compare it to 'BLAH'.

如果实在要用, 可以函数用在后面

```sql
WHERE Field = INVERSE_FUNCTION('BLAH')
```



### 5.3 BETWEEN vs <= and >=

建议少使用BETWEEN, 因为不同的 sql 实现可能不一样

+ between 的范围是包含两边的边界值
  eg： id between 3 and 7 等价与 id >=3 and id<=7

+ not between 的范围是不包含边界值
  eg：id not between 3 and 7 等价与 id < 3 or id>7



# 6. 参考资料:

+ https://blog.csdn.net/shiyong1949/article/details/80923083
+ https://www.journaldev.com/19165/sql-in-sql-not-in
+ https://stackoverflow.com/questions/799584/what-makes-a-sql-statement-sargable
+ https://stackoverflow.com/questions/173041/not-in-vs-not-exists
+ https://www.polderknowledge.nl/2018/03/02/sql-beware-null-where-not/
+ https://yefeihonours.github.io/post/mysql/in_and_exists/

