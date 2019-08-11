---
title: "linux下动态库查找路径及LD_LIBRARY_PATH问题"
date: 2019-08-11 19:02:59
tags:
- linux
---

1



### 说到和动态库查找路径相关的问题，总体上可以分为两类：

第一类：通过源代码编译程序时出现的找不到某个依赖包的问题，而如果此时你恰好已经按照它的要求确确实实、千真万确、天地良心地把依赖库给装好了，它还给你耍混、犯二，有一股折腾不死人不偿命的劲儿，那让人真是气儿不打一处来，如果Linux此时有头有脸，你是不是特想抽它丫两大嘴巴；
   第二类：就是在运行程序的时候，明明把那个程序需要的依赖包都已经安装的妥妥的了，可运行的时候人家就告诉你说“error while loading shared libraries: **libxxx****.so.y**: cannot open shared object file: No such file or directory”，任凭你怎么折腾都没用。此时你要是心想“撤吧，哥们，Linux太TM欺负人了，不带这么玩儿的”，那你就大错特错了，只要你抱着“美好的事情的总会发生”和“办法永远比问题多”的信念坚持下去，你就一定会成功。话的意思有点自欺欺人，精神鸦片的味道在里面，但确实是这么个理儿。

<!-- more -->

###    问题1：通过源代码安装程序

通过源码包安装程序时，主要用到了“三大步”策略：configure、make和make install 。出问题最多的就是在configure阶段，很多初学者由于不知道configure的那么多参数该怎么用，所以往往为了省事，一句简单的“./configure”下去，百分之八九十都能成功，可问题往往就出在剩下的百分之十几上面了。



在安装的configure阶段，为了检测安装安装环境是否满足，通常情况下都是通过一个叫做pkg-config的工具来检测它需要依赖的动态库是否存在，这个工具我们在上一篇博文已经认识过了。pkg-config通常情况都是位于/usr/bin目录下，是个可执行程序。在configure阶段，通常都会用pkg-config来判断所依赖的动态库是否存在。现在问题就是，这个工具是如何判断的呢？它的依据是什么？当这两个问题弄明白了，真相也就大白了。



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







### 参考资料

+ http://blog.chinaunix.net/uid-23069658-id-4028681.html