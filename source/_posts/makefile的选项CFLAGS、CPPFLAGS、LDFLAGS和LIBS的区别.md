---
title: "makefile的选项CFLAGS、CPPFLAGS、LDFLAGS和LIBS的区别"
date: 2018-10-9 18:34:02
tags:
- makefile
- linux
---



先看一个例子:

```shell
export CFLAGS="-I/root/ARM/opt/include"
export LDFLAGS="-L/root/ARM/opt/lib"
```



**CFLAGS**： 指定头文件（.h文件）的路径，如：CFLAGS=-I/usr/include -I/path/include。同样地，安装一个包时会在安装路径下建立一个include目录，当安装过程中出现问题时，试着把以前安装的包的include目录加入到该变量中来。

**LDFLAGS**：gcc 等编译器会用到的一些优化参数，也可以在里面指定库文件的位置。用法：LDFLAGS=-L/usr/lib -L/path/to/your/lib。每安装一个包都几乎一定的会在安装目录里建立一个lib目录。如果明明安装了某个包，而安装另一个包时，它愣是说找不到，可以抒那个包的lib路径加入的LDFALGS中试一下。

**LIBS**：告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv

<!-- more -->

### CFLAGS,CXXFLAGS,CPPFLAGS的区别

CFLAGS 表示用于 C 编译器的选项

CXXFLAGS 表示用于 C++ 编译器的选项。

CPPFLAGS 可以 用于 C 和 C++ 两者。



### LDFLAGS,LIBS的区别

LDFLAGS是选项，LIBS是要链接的库。都是喂给ld的，只不过一个是告诉ld怎么吃，一个是告诉ld要吃什么。

看看如下选项：

```shell
LDFLAGS = -L/var/xxx/lib -L/opt/mysql/lib
LIBS = -lmysqlclient -liconv
```

这就明白了。LDFLAGS告诉链接器从哪里寻找库文件，LIBS告诉链接器要链接哪些库文件。不过使用时链接阶段这两个参数都会加上，所以你即使将这两个的值互换，也没有问题。



说到这里，进一步说说LDFLAGS指定-L虽然能让链接器找到库进行链接，但是运行时链接器却找不到这个库，如果要让软件运行时库文件的路径也得到扩展，那么我们需要增加这两个库给"-Wl,R"

```
LDFLAGS = -L/var/xxx/lib -L/opt/mysql/lib -Wl,R/var/xxx/lib -Wl,R/opt/mysql/lib
```

如 果在执行./configure以前设置环境变量export LDFLAGS="-L/var/xxx/lib -L/opt/mysql/lib -Wl,R/var/xxx/lib -Wl,R/opt/mysql/lib" ，注意设置环境变量等号两边不可以有空格，而且要加上引号哦（shell的用法）。执行configure以后，Makefile将会设置这个选项， 链接时会有这个参数，编译出来的可执行程序的库文件搜索路径就得到扩展了。



### 参考链接:

https://forum.golangbridge.org/t/cflags-ldflags-documentation-somewhere/4520/10