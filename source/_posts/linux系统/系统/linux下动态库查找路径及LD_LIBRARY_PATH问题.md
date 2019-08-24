---
title: 动态库查找路径及LD_LIBRARY_PATH问题
tags:
  - linux
categories:
  - linux系统
  - 系统
abbrlink: 17c0913d
date: 2019-08-17 13:52:59
---

说到和动态库查找路径相关的问题，总体上可以分为两类：

+ 第一类：通过源代码编译程序时出现的找不到某个依赖包的问题
+ 第二类：就是在运行程序的时候，明明把那个程序需要的依赖包都已经安装的妥妥的了，可运行的时候人家就告诉你说`error while loading shared libraries: libxxx.so.y: cannot open shared object file: No such file or directory`。

<!-- more -->

###    1. 源代码安装程序找不到依赖库

通过源码包安装程序时，主要用到了“三大步”策略：configure、make和make install 。出问题最多的就是在configure阶段，很多初学者由于不知道configure的那么多参数该怎么用，所以往往为了省事，一句简单的“./configure”下去，百分之八九十都能成功，可问题往往就出在剩下的百分之十几上面了。



在安装的configure阶段，为了检测安装安装环境是否满足，通常情况下都是通过一个叫做`pkg-config`的工具来检测它需要依赖的动态库是否存在，这个工具我们在上一篇博文已经认识过了。pkg-config通常情况都是位于/usr/bin目录下，是个可执行程序。在configure阶段，通常都会用pkg-config来判断所依赖的动态库是否存在。现在问题就是，这个工具是如何判断的呢？它的依据是什么？当这两个问题弄明白了，真相也就大白了。



 一般当我们安装完某个程序后，如果它提供了动态库的功能，在源码中都会有一个或多个以pc结尾的文件，当执行完make install后这些pc文件拷贝到${prefix}/lib/pkgconfig这个目录里，这里的prefix就是我们在configure阶段时通过配置参数--prefix指定的，缺省情况这个值就是/usr/local，所以这些pc文件最终会被拷贝到/usr/local/lib/pkgconfig目录下。可能有人会问，这些pc文件有啥用呢？我们随便打开一个来瞅瞅：

```shell

[root@localhost ~]# cat /usr/local/lib/pkgconfig/librtmp.pc
prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
incdir=${prefix}/include


Name: librtmp
Description: RTMP implementation
Version: v2.3
Requires: libssl,libcrypto
URL: http://rtmpdump.mplayerhq.hu
Libs: -L${libdir} -lrtmp -lz
Cflags: -I${incdir}
```

跟我们configure阶段相关的主要集中在Libs和Cflags两项上面，如果你此时再执行下面这两条命令，就全明白了：

```shell
[root@localhost ~]# pkg-config --cflags librtmp
-I/usr/local/include
[root@localhost ~]# pkg-config --libs librtmp
-L/usr/local/lib -lrtmp -lz -lssl -lcrypto
```

也就是说，pkg-config把我们以前需要在Makefile里指定编译和链接时所需要用到的参数从手工硬编码的模式变成了自动完成，节约了多少跨平台移植的兼容性问题。



##### 1.1 安装了找不到依赖库的原因

假如说，如果我们将要的编译的软件包依赖librtmp这个动态库，那么此时在我系统上这个检测就算通过了。当然这只是第一步，检测过了不一定兼容，这里我们只讨论能不能找到依赖库的问题。好了，如果说找不到某个库该怎么办。前提是你确确实实已经安装了它需要的库，不用多想，原因只有一个，pkg-config找不到这个与这个库对应的pc文件。

为什么会找不到呢，原因又有两点：

1、pkg-config搜索了所有它认为合适的目录都没找着这个库对应的pc文件的下落；

2、这个库在发布时根本就没有提供它的pc文件。

> 那么pkg-config的查找路径是哪里？

pkg-config较老的版本里，缺省情况下会到/usr/lib/pkgconfig、/usr/loca/lib/pkgconfig、/usr/share/pkgconfig等目录下去搜索pc文件，据我所知在0.23以及之后的版本里pkg-config的源码里已经没有关于缺省搜索路径的任何硬编码的成分了，取而代之的是，当你看pkg-config的man手册时会有下面一段话：

```
pkg-config retrieves information about packages from special metadata files. These files are  named  after the  package,  with  the extension .pc.
By default, pkg-config looks in the directory ___prefix___/lib/pkgconfig for these files; it will also look in the colon-separated (on Windows, semicolon-separated) list of directories specified by the PKG_CONFIG_PATH environment variable.



PKG_CONFIG_PATH
    A colon-separated (on Windows, semicolon-separated) list of directories to search  for  .pc  files. The  default directory will always be searched after searching the path; the default is ___libdir___/pkg-config:___datadir___/pkgconfig where libdir is the libdir where pkg-config and  datadir  is  the  datadir where pkg-config was installed.
```



  上面的prefix、libdir和datadir，就是安装pkg-config时被设定好的，具体情况是：

+ 如果你是通过yum和rpm包安装的

  ```bash
  prefix=/usr
  libdir=${prefix}/lib=/usr/lib
  datadir=${prefix}/share=/usr/share
  ```

+ 如果你是通过源码包安装的，且没有指定prefix的值

  ```bash
  prefix=/usr/local
  libdir=${prefix}/lib=/usr/local/lib
  datadir=${prefix}/share=/usr/local/share 
  ```



pkg-config在查找对应软件包的信息时的缺省搜索路径已经很清楚了，就是是`${libdir}/pkgconfig`和`${datadir}/pkgconfig`。如果你软件包对应的pc文件都不在这两个目录下时，pkg-config肯定找不到。既然原因都已经找到了，那解决办法也就多种多样了。



##### 1.2 解决找不到库的问题(PKG_CONFIG_PATH)

+ 我们可以在安装我们那个被依赖的软件包时，在configure阶段用--prefix参数把安装目录指定到/usr目录下；

+ 也可以按照上面说的，通过一个名叫`PKG_CONFIG_PATH`的环境变量来向pkg-config指明我们自己的pc文件所在的路径，不过要注意的是`PKG_CONFIG_PATH`所指定的路径优先级比较高，pkg-config会先进行搜索，完了之后才是去搜索缺省路径。

前者的优点是以后再通过源码安装软件时少了不少麻烦，缺点是用户自己的软件包和系统软件混到一起不方便管理，所以实际使用中，后者用的要多一些。如下:

```bash
export PKG_CONFIG_PATH=/your/local/path:$PKG_CONFIG_PATH
```

  然后，在configure时就绝对没问题了。



### 2. 程序运行时出现libxxx.so.y => not found

##### 2.1 ldd 查看依赖的动态库

用`ldd 可执行程序名`可以查看一个软件启动时所依赖的动态库，如果输出项有“libxxx.so.y=> not found”一项，你这个软件100%运行不起来。我们来做个试验：

```bash
[root@localhost ~]# echo $LD_LIBRARY_PATH    //嘛也没有
[root@localhost ~]# ldd /usr/local/bin/ffmpeg
........
libmp3lame.so.0 => /usr/local/lib/libmp3lame.so.0 (0x0088c000)
libfaac.so.0 => /usr/local/lib/libfaac.so.0 (0x00573000)
........
```



我的系统里没有设置`LD_LIBRARY_PATH`环境变量，现在我们把其中的一个库`libmp3lame.so.0`从`/usr/loca/lib`下移动到/opt目录里，并执行ldconfig，让`libmp3lame.so.0`彻底从`/etc/ld.so.cache`里面消失。其实`libmp3lame.so.0`只是`libmp3lame.so.0.0.0`的一个符号链接，我们真正需要移动的是后者.

完了之后再执行ldd /usr/local/bin/ffmpeg时结果如下：

```bash
[root@localhost ~]# ldd /usr/local/bin/ffmpeg
........
libmp3lame.so.0 => not found    //果然Not found 了
libfaac.so.0 => /usr/local/lib/libfaac.so.0 (0x004a4000)
........

[root@localhost ~]# ffmpeg --help
ffmpeg: error while loading shared libraries: libmp3lame.so.0: cannot open shared object file: No such file or directory  //此时ffmpeg当然运行不起来
```

   

##### 2.2 LD_LIBRARY_PATH 

我们来试试LD_LIBRARY_PATH，看看好使不：

```bash
[root@localhost opt]# export LD_LIBRARY_PATH=/opt:$LD_LIBRARY_PATH
[root@localhost opt]# ldd /usr/local/bin/ffmpeg
........
libmp3lame.so.0 => not found           //纳尼？？！！！
libfaac.so.0 => /usr/local/lib/libfaac.so.0 (0x00124000)
........
```

还记得上面提到了软链接么，`libmp3lame.so.0`就是`libmp3lame.so.0.0.0`的软链接，这是动态库的命名规范的一种公约，我们只要在/opt/目录下建立一个名为`libmp3lame.so.0`的到`/opt/libmp3lame.so.0.0.0`的软链接就OK了：

```bash
[root@localhost opt]# ln -s libmp3lame.so.0.0.0 libmp3lame.so.0
[root@localhost opt]# ldd /usr/local/bin/ffmpeg
........
libmp3lame.so.0 => /opt/libmp3lame.so.0 (0x00767000)   //终于圆满了:)
libfaac.so.0 => /usr/local/lib/libfaac.so.0 (0x006e8000)
........
```

### 3. 总结

针对动态库路径查找的种种问题，无非就这么两大类，关键是找对原因，对症下药，方能药到病除。

+ PKG_CONFIG_PATH从字面意思上翻译，就是“软件包的配置路径”，这不很明显了么，编译软件时如果出现找不到所依赖的动态库时都全靠PKG_CONFIG_PATH了；
+ LD_LIBRARY_PATH也很直白了“装载器的库路径”，LD是Loader的简写，在Linux系统启动一个程序的过程就叫做装载，一个程序要执行时它或多或少的会依赖一些动态库(静态编译的除外)。

### 4. 参考资料

+ http://blog.chinaunix.net/uid-23069658-id-4028681.html
+ https://prefetch.net/articles/linkers.badldlibrary.html