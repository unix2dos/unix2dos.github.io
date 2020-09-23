---
title: "sql常见基础知识"
date: 2020-09-23 20:00:00
tags:
- sql
- mysql
---


# 1. mysql 数值数据类型

MySQL支持所有标准的SQL数值数据类型。这些类型包括精确的数字数据类型（INTEGER、SMALLINT、DECIMAL和NUMERIC，以及近似数字数据类型（FLOAT、REAL和DOUBLE PRECISION）。

+ INT是INTEGER的同义词。

+ DEC, FIXED, NUMERIC是DECIMAL的同义词。

+ DOUBLE视为DOUBLE PRECISION（非标准扩展）的同义词。

+ REAL视为DOUBLE PRECISION（非标准变体）的同义词，除非启用REAL_AS_FLOAT SQL模式。

从MySQL8.0.17开始，ZEROFILL属性不推荐用于数值数据类型，在未来的MySQL版本中，对它的支持将被删除。

<!-- more -->

### 1.1 整数

对于整数数据类型，M表示最大显示宽度。最大显示宽度为255。显示宽度与类型可以存储的值范围无关。

| Type      | Storage (Bytes) | Minimum Value Signed | Minimum Value Unsigned | Maximum Value Signed | Maximum Value Unsigned |
| --------- | --------------- | -------------------- | ---------------------- | -------------------- | ---------------------- |
| TINYINT   | 1               | -128                 | 0                      | 127                  | 255                    |
| SMALLINT  | 2               | -32768               | 0                      | 32767                | 65535                  |
| MEDIUMINT | 3               | -8388608             | 0                      | 8388607              | 16777215               |
| INT       | 4               | -2147483648          | 0                      | 2147483647           | 4294967295             |
| BIGINT    | 8               | -2^63                | 0                      | 2^63 -1              | 2^64 -1                |

+ [TINYINT[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

A very small integer. The signed range is -128 to 127. The unsigned range is 0 to 255.



+ [BOOL](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html), [BOOLEAN](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

These types are synonyms for [TINYINT(1)](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html). A value of zero is considered false. Nonzero values are considered true



+ [SMALLINT[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

A small integer. The signed range is -32768 to 32767. The unsigned range is 0 to 65535.



+ [MEDIUMINT[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

A medium-sized integer. The signed range is -8388608 to 8388607. The unsigned range is 0 to 16777215.



+ [INT[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

  [INTEGER[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

A normal-size integer. The signed range is -2147483648 to 2147483647. The unsigned range is 0 to 4294967295.



+ [BIGINT[(**M**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

A large integer. The signed range is -9223372036854775808 to 9223372036854775807. The unsigned range is 0 to 18446744073709551615.



### 1.2 小数

对于浮点和定点数据类型，M是可存储的总位数。

+ 第一类

[DECIMAL[(**M**[,**D**\])] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)

[DEC[(**M**[,**D**\])] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)

[NUMERIC[(**M**[,**D**\])] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)

[FIXED[(**M**[,**D**\])] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)



M是总位数（精度），D是小数点后的位数（刻度）。十进制的最大位数（M）为65。支持的最大小数位数（D）为30。

如果省略M，默认值为10。如果省略D，默认值为0。

小数点和（对于负数）符号不计算在M中。如果D为0，则值没有小数点或小数部分。

从MySQL 8.0.17开始，DECIMAL类型的列（以及任何同义词）都不推荐使用UNSIGNED属性，在将来的MySQL版本中，将删除对它的支持。



+ 第二类

[FLOAT[(**M**,**D**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)



M是总位数，D是小数点后的位数。如果省略M和D，则值存储在硬件允许的范围内。单精度浮点数精确到小数点后7位左右。

FLOAT（M，D）是一个非标准的MySQL扩展。从MySQL 8.0.17开始，不推荐使用这种语法，在将来的MySQL版本中将删除对它的支持。



+ 第三类

[DOUBLE[(**M**,**D**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)

[DOUBLE PRECISION[(**M**,**D**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)

[REAL[(**M**,**D**)\] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)



M是总位数，D是小数点后的位数。如果省略M和D，则值存储在硬件允许的范围内。双精度浮点数精确到小数点后15位左右。DOUBLE（M，D）是一个非标准的MySQL扩展。从MySQL 8.0.17开始，不推荐使用这种语法，在将来的MySQL版本中将删除对它的支持。



+ 第四类

[FLOAT(**p**) [UNSIGNED\] [ZEROFILL]](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)



如果p从0到24，数据类型变为FLOAT，没有M或D值。如果p从25到53，数据类型变为DOUBLE，没有M或D值。



### 1.3 总结

+ 浮点数不推荐使用 FLOAT 和 DOUBLE
+ 浮点数推荐使用DECIMAL, 不推荐使用无符号



# 2. int(11)  括号的数字

对于整数数据类型，M表示最大显示宽度。最大显示宽度为255。显示宽度与类型可以存储的值范围无关。

### 2.1 int(1)、tinyint(4) 哪个大?

int 大。

注意数字类型后面括号中的数字，不表示长度，表示的是显示宽度，这点与 varchar、char 后面的数字含义是不同的。

也就是说不管 int 后面的数字是多少，它存储的范围始终是 -2^31 到 2^31 - 1，但是int(1)只显示个位数。

综上整型的数据类型括号内的数字不管是多少，所占的存储空间都是一样。



# 3. char(4) 和 varchar(4) 括号的数字

CHAR和VARCHAR类型声明的长度指示要存储的最大字符数。

例如，CHAR（30）最多可容纳30个字符。如果是utf8mb4格式, 最多可容纳30个字符或汉字。

### 3.1 长度限制

CHAR列的长度固定为创建表时声明的长度。长度可以是0到255之间的任何值。

VARCHAR列中的值是可变长度的字符串。长度可以指定为0到65535之间的值。VARCHAR的有效最大长度取决于最大行大小和使用的字符集。



### 3.2 填空和截断

当存储CHAR值时，将使用指定长度的空格对其进行右填充。

VARCHAR值在存储时不填充。在存储和检索值时，尾部空格将被保留，这与标准SQL一致。



如果未启用strict SQL模式，并且将值分配给超过该列最大长度的CHAR或VARCHAR列，则会截断该值以适合该列，并生成警告。

对于非空格字符的截断，可以使用strict SQL模式导致发生错误（而不是警告）并禁止插入值。



### 3.3 示例

下表通过显示将各种字符串值存储到CHAR（4）和VARCHAR（4）列中的结果（假设列使用单字节字符集，如latin1），说明了CHAR和VARCHAR之间的区别。

| Value      | CHAR(4) | Storage Required | VARCHAR(4) | Storage Required |
| ---------- | ------- | ---------------- | ---------- | ---------------- |
| ''         | '  '    | 4 bytes          | ''         | 1 byte           |
| 'ab'       | 'ab '   | 4 bytes          | 'ab'       | 3 bytes          |
| 'abcd'     | 'abcd'  | 4 bytes          | 'abcd'     | 5 bytes          |
| 'abcdefgh' | 'abcd'  | 4 bytes          | 'abcd'     | 5 bytes          |



# 4. count(*), count(1) 的区别



### 4.1 count(*), count(1), count(column)

- COUNT(*) counts all rows
- COUNT(column) counts non-NULLs only
- [COUNT(1) is the same as COUNT(*)](https://stackoverflow.com/a/1221649/27535) because 1 is a non-null expressions

Your use of COUNT(*) or COUNT(column) should be based on the desired output *only*.



### 4.2 count(*), count(1)

Same IO, same plan, the works



# 5. TODO:

索引下推

开窗函数

子查询效率问题



# 6. 参考资料

+ https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html
+ https://dev.mysql.com/doc/refman/8.0/en/char.html
+ https://stackoverflow.com/a/3003533