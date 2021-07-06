---
title: grep命令的使用介绍
tags:
  - linux
categories:
  - 2-linux系统
  - 系统
abbrlink: 6dc03659
date: 2020-03-16 00:00:00
---

grep是Linux中最常用的”文本处理工具”之一，grep与sed、awk合称为Linux中的三剑客。

我们可以使用grep命令在文本中查找指定的字符串，就像打开txt文件，使用 “Ctrl+F” 在文本中查找某个字符串一样，说白了，可以把grep理解成字符查找工具。

<!-- more -->

# 1. 介绍

grep的全称为： **G**lobal search **R**egular **E**xpression and **P**rint out the line, 表示全局正则表达式版本，它的使用权限是所有用户。

`grep`的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。

### 1.1 实例

```bash
# 查找指定进程
ps -ef|grep python 

# 查找指定进程个数
ps -ef|grep -c python

# 从文件中查找关键词
grep 'linux' file1.txt

# 找出已w开头的行内容
cat file1.txt |grep ^w

# 找出非w开头的行内容
cat file2.txt |grep ^[^w]

# 输出以hat结尾的行内容
cat test.txt |grep hat$

# 在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件
grep test *file 

# 以递归的方式查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件
grep -r update /etc/acpi 
```



# 2. 使用

cat a.txt, 以下面的文件为测试文件

```txt
test
a
b
c
d
e
f
g
test1
TEST2
```



### 2.1 基础搜索

我们先来一个最基础的搜索:

```bash
grep "test" a.txt

test
test1
```



如果我们想要在搜索字符串的时候，不区分大小写，应该怎样做呢？grep很贴心，为我们准备了一个选项，使用”-i”选项，即可在搜索时不区分大小写，示例如下：

```bash
grep -i "test" a.txt

test
test1
TEST2
```



我们还想要知道哪行文本包含”test”字符串，则可以使用”-n”选项，表示显示打印出的行在文本中的行号，示例如下

```bash
grep -in "test" a.txt

1:test
9:test1
10:TEST2
```



被匹配到的关键字没有高亮显示，如果我们想要高亮显示行中的关键字，该怎么办呢？我们可以使用”–color”选项，高亮显示行中的关键字，示例如下

```bash
grep -in --color "test" a.txt

# 其实结果一样, 因为在 mac 下, grep 已经是别名, 我们可以输出一下

alias grep
grep='grep --color=auto --exclude-dir={.bzr,CVS,.git,.hg,.svn,.idea,.tox}'
```



如果我们只想知道有多少行包含指定的字符串，而不在乎哪些行包含这些字符串，我们可以使用-c，获取到符合条件的总行数。

```bash
grep -ic "test" a.txt

3
```



如果我们只想看被匹配到的关键字，不想整行都被打印出来，可以吗？必须的，使用”-o”选项即可只打印出匹配到的关机字，而不打印出整行，示例如下。

```bash
grep -ino "test" a.txt

1:test
9:test
10:TEST
```

但是需要注意，”-o”选项会把每个匹配到的关键字都单独显示在一行中进行输出(即一行有2个匹配会输出两行)



如果我们搜索内容, 但结果只想展示文件名, 可以用`-l`选项

```bash
grep -l "test" a.txt

a.txt
```



如果我们想搜索文件的名字(注意不是内容), 需要用到 `find`指令

``` bash
find <path> -name *FileName*
```



### 2.2 高级搜索

我们在使用grep命令搜索文本时，往往有这种需求：在找到对应的关键字时，同时需要显示关键字附近的信息

 例如我们先找字母c

```bash
grep -in "c" a.txt

4:c
```

不仅找字母 c, 还找附近的行, 我们可以使用”-B”选项，显示符合条件的行之前的行，”B”有before之意，示例如下

```bash
grep -in -B 3 "c" a.txt

1-test
2-a
3-b
4:c
```

与”-B”选项对应的选项是”-A”选项，”-B”有Before之意，”-A”有After之意，聪明如你，一定已经猜到了”-A”的含义，没错，”-A”代表显示符合条件的行的同时，还要显示之后的行，”-A3″表示同时显示符合条件的行之后的3行。



说了”-A”，说了”-B”，现在说说”-C”，”-C”选项可以理解为”-A与-B”的结合，”-C”选项表示在显示符合条件的行的同时，也会显示其前后的行，如”-C1″，”-C1″表示打印符合条件的行的同时，也打印出之前的一行与之后的一行，”-C”有Context之意（上下文之意），示例如下。

```bash
grep -in -C 3 "c" a.txt

1-test
2-a
3-b
4:c
5-d
6-e
7-f
```



精确匹配，就是”test”作为一个独立的单词存在，而不是包含于某个字符串中，那么，如果有这种需求，我们怎么办呢？使用”-w”选项可以实现我们的需求，示例如下。

```bash
grep -inw test a.txt #”-w”有word之意，表示搜索的字符串作为一个独立的单词时才会被匹配到。

1:test
```

有的时候，我们需要反向查找，比如，查找”不包含某个字符串”的行，这个时候，我们需要用到”-v”选项，示例如下。

```bash
grep -iv test a.txt

a
b
c
d
e
f
g
```



我们也可以同时在文本中搜索了”a”字符串与”b”字符串，包含这两个字符串中任意一个的行都会被打印出来，没错，就像上图中的示例一样，使用”-e”选项可以同时匹配多个目标，多个目标之间存在”或”关系，即匹配其中的任意一个都算作匹配成功

```bash
grep -ie a -ie b a.txt

a
b
```



### 2.3 正则搜索

在使用”-E”选项时，grep才支持”扩展正则表达式”，不使用”-E”选项时，grep默认只支持”基本正则表达式”。
我们在使用grep时，可以使用”-P”选项，指明使用perl兼容的正则表达式。

```bash
grep -iE "a|b" a.txt

a
b
```

其实，除了grep命令，其实还有egrep命令，还有fgrep命令（fast grep），它们有各自的特点。
grep：支持基本正则表达式
egrep：支持扩展正则表达式，相当于grep -E
fgrep：不支持正则表达式，只能匹配写死的字符串，但是速度奇快，效率高，fastgrep



### 2.4 总结

-i：在搜索的时候忽略大小写
-n：显示结果所在行号
-c：统计匹配到的行数，注意，是匹配到的总行数，不是匹配到的次数
-o：只显示符合条件的字符串，但是不整行显示，每个符合条件的字符串单独显示一行
-v：输出不带关键字的行（反向查询，反向匹配）
-w：匹配整个单词，如果是字符串中包含这个单词，则不作匹配
-Ax：在输出的时候包含结果所在行之后的指定行数，这里指之后的x行，A：after
-Bx：在输出的时候包含结果所在行之前的指定行数，这里指之前的x行，B：before
-Cx：在输出的时候包含结果所在行之前和之后的指定行数，这里指之前和之后的x行，C：context
-e：实现多个选项的匹配，逻辑or关系
-P：表示使用兼容perl的正则引擎。
-E：使用扩展正则表达式，而不是基本正则表达式，在使用”-E”选项时，相当于使用egrep。
-l :   搜索结果只显示文件名


# 3. 参考资料

+ https://www.zsythink.net/archives/1733
+ https://www.yiibai.com/linux/grep.html