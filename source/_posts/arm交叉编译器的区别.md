---
title: "arm交叉编译器的区别"
date: 2018-10-08 16:33:11
tags:
- arm
- linux
---



## 为什么要用交叉编译器？

交叉编译通俗地讲就是在一种平台上编译出能运行在体系结构不同的另一种平台上的程序，比如在PC平台（X86 CPU）上编译出能运行在以ARM为内核的CPU平台上的程序，编译得到的程序在X86 CPU平台上是不能运行的，必须放到ARM CPU平台上才能运行，虽然两个平台用的都是Linux系统。

交叉编译工具链是一个由编译器、连接器和解释器组成的综合开发环境，交叉编译工具链主要由binutils、gcc和glibc三个部分组成。有时出于减小 libc 库大小的考虑，也可以用别的 c 库来代替 glibc，例如 uClibc、dietlibc 和 newlib。

建立交叉编译工具链是一个相当复杂的过程，如果不想自己经历复杂繁琐的编译过程，网上有一些编译好的可用的交叉编译工具链可以下载，但就以学习为目的来说读者有必要学习自己制作一个交叉编译工具链（目前来看，对于初学者没有太大必要自己交叉编译一个工具链）。

<!-- more -->

## 分类和说明

从授权上，分为免费授权版和付费授权版。

免费版目前有三大主流工具商提供，第一是GNU（提供源码，自行编译制作），第二是 Codesourcery，第三是Linora。

收费版有ARM原厂提供的armcc、IAR提供的编译器等等，因为这些价格都比较昂贵，不适合学习用户使用，所以不做讲述。



+ arm-none-linux-gnueabi-gcc：是 Codesourcery 公司（目前已经被Mentor收购）基于GCC推出的的ARM交叉编译工具。可用于交叉编译ARM（32位）系统中所有环节的代码，包括裸机程序、u-boot、Linux  kernel、filesystem和App应用程序。

+ arm-linux-gnueabihf-gcc：是由 Linaro 公司基于GCC推出的的ARM交叉编译工具。可用于交叉编译ARM（32位）系统中所有环节的代码，包括裸机程序、u-boot、Linux kernel、filesystem和App应用程序。

+ aarch64-linux-gnu-gcc：是由 Linaro 公司基于GCC推出的的ARM交叉编译工具。可用于交叉编译ARMv8 64位目标中的裸机程序、u-boot、Linux  kernel、filesystem和App应用程序。

+ arm-none-elf-gcc：是 Codesourcery 公司（目前已经被Mentor收购）基于GCC推出的的ARM交叉编译工具。可用于交叉编译ARM MCU（32位）芯片，如ARM7、ARM9、Cortex-M/R芯片程序。

+ arm-none-eabi-gcc：是 GNU 推出的的ARM交叉编译工具。可用于交叉编译ARM MCU（32位）芯片，如ARM7、ARM9、Cortex-M/R芯片程序。



## 交叉编译器下载

- arm-none-linux-gnueabi-gcc下载：<http://www.veryarm.com/arm-none-linux-gnueabi-gcc>
- arm-linux-gnueabihf-gcc下载：<http://www.veryarm.com/arm-linux-gnueabihf-gcc>
- aarch64-linux-gnu-gcc下载：<http://www.veryarm.com/aarch64-linux-gnu-gcc>
- arm-none-elf-gcc下载：<http://www.veryarm.com/arm-none-elf-gcc>
- arm-none-eabi-gcc下载：<http://www.veryarm.com/arm-none-eabi-gcc>

 以上地址都是直接从官网转存到百度云盘，仅为方便国内用户下载使用，并非本站制作，请勿用于商业或者非法用途。因为版本多难以选择，所以我们建议您使用该类编译器的最新版本。



## 命名规则

交叉编译工具链的命名规则为：`arch [-vendor][-os] [-(gnu)eabi]`

- **arch** - 体系架构，如ARM，MIPS
- **vendor** - 工具链提供商
- **os** - 目标操作系统
- **eabi** - 嵌入式应用二进制接口（Embedded Application Binary Interface）

根据对操作系统的支持与否，ARM GCC可分为支持和不支持操作系统，如

- **arm-none-eabi**：这个是没有操作系统的，自然不可能支持那些跟操作系统关系密切的函数，比如fork(2)。他使用的是newlib这个专用于嵌入式系统的C库。
- **arm-none-linux-eabi**：用于Linux的，使用Glibc



##  实例

#### 1、arm-none-eabi-gcc

（ARM architecture，no vendor，not target an operating system，complies with the ARM EABI）
用于编译 ARM 架构的裸机系统（包括 ARM Linux 的 boot、kernel，不适用编译 Linux 应用 Application），一般适合 ARM7、Cortex-M 和 Cortex-R 内核的芯片使用，所以不支持那些跟操作系统关系密切的函数，比如fork(2)，他使用的是 newlib 这个专用于嵌入式系统的C库。

#### 2、arm-none-linux-gnueabi-gcc  //常用的

(ARM architecture, no vendor, creates binaries that run on the **Linux** operating system, and uses the GNU EABI)

主要用于基于ARM架构的Linux系统，可用于编译 ARM 架构的 u-boot、Linux内核、linux应用等。arm-none-linux-gnueabi基于GCC，使用Glibc库，经过 `Codesourcery` 公司优化过推出的编译器。arm-none-linux-gnueabi-xxx 交叉编译工具的浮点运算非常优秀。一般ARM9、ARM11、Cortex-A 内核，带有 Linux 操作系统的会用到。

#### 3、arm-eabi-gcc

Android ARM 编译器。

#### 4、armcc

ARM 公司推出的编译工具，功能和 arm-none-eabi 类似，可以编译裸机程序（u-boot、kernel），但是不能编译 Linux 应用程序。armcc一般和ARM开发工具一起，Keil MDK、ADS、RVDS和DS-5中的编译器都是armcc，所以 armcc 编译器都是收费的（爱国版除外，呵呵~~）。

#### 5、arm-none-uclinuxeabi-gcc 和 arm-none-symbianelf-gcc

arm-none-uclinuxeabi 用于**uCLinux**，使用Glibc。

arm-none-symbianelf 用于**symbian**，没用过，不知道C库是什么 。



## ABI 和 EABI

**ABI**：二进制应用程序接口(Application Binary Interface (ABI) for the ARM Architecture)。在计算机中，应用二进制接口描述了应用程序（或者其他类型）和操作系统之间或其他应用程序的低级接口。

**EABI**：嵌入式ABI。嵌入式应用二进制接口指定了文件格式、数据类型、寄存器使用、堆积组织优化和在一个嵌入式软件中的参数的标准约定。开发者使用自己的汇编语言也可以使用 EABI 作为与兼容的编译器生成的汇编语言的接口。

两者主要区别是，ABI是计算机上的，EABI是嵌入式平台上（如ARM，MIPS等）。



## Codesourcery公司

Codesourcery推出的产品叫Sourcery G++ Lite Edition，其中基于command-line的编译器是免费的，在官网上可以下载，而其中包含的IDE和debug 工具是收费的，当然也有30天试用版本的。

目前CodeSourcery已经由明导国际(Mentor Graphics)收购，所以原本的网站风格已经全部变为 Mentor 样式，但是 Sourcery G++ Lite Edition 同样可以注册后免费下载。

Codesourcery一直是在做ARM目标 GCC 的开发和优化，它的ARM GCC在目前在市场上非常优秀，很多 patch 可能还没被gcc接受，所以还是应该直接用它的。

而且他提供Windows下[mingw交叉编译的]和Linux下的二进制版本，比较方便；如果不是很有时间和兴趣，不建议下载 src 源码包自己编译，很麻烦，Codesourcery给的shell脚本很多时候根本没办法直接用，得自行提取关键的部分手工执行，又费精力又费时间，如果想知道细节，其实不用自己编译一遍，看看他是用什么步骤构建的即可，如果你对交叉编译器感兴趣的话。



## arm-linux-gnueabi-gcc 和 arm-linux-gnueabihf-gcc

两个交叉编译器分别适用于 armel 和 armhf 两个不同的架构，armel 和 armhf 这两种架构在对待浮点运算采取了不同的策略（有 fpu 的 arm 才能支持这两种浮点运算策略）。

其实这两个交叉编译器只不过是 gcc 的选项 -mfloat-abi 的默认值不同。gcc 的选项 -mfloat-abi 有三种值 **soft、softfp、hard**（其中后两者都要求 arm 里有 fpu 浮点运算单元，soft 与后两者是兼容的，但 softfp 和 hard 两种模式互不兼容）：
**soft：** 不用fpu进行浮点计算，即使有fpu浮点运算单元也不用，而是使用软件模式。
**softfp：** armel架构（对应的编译器为 arm-linux-gnueabi-gcc ）采用的默认值，用fpu计算，但是传参数用普通寄存器传，这样中断的时候，只需要保存普通寄存器，中断负荷小，但是参数需要转换成浮点的再计算。
**hard：** armhf架构（对应的编译器 arm-linux-gnueabihf-gcc ）采用的默认值，用fpu计算，传参数也用fpu中的浮点寄存器传，省去了转换，性能最好，但是中断负荷高。



把以下测试使用的C文件内容保存成 mfloat.c：

```c
#include <stdio.h>
int main(void)
{
    double a,b,c;
    a = 23.543;
    b = 323.234;
    c = b/a;
    printf(“the 13/2 = %f\n”, c);
    printf(“hello world !\n”);
    return 0;
}
```



**1、使用 arm-linux-gnueabihf-gcc 编译，使用“-v”选项以获取更详细的信息：**
\# arm-linux-gnueabihf-gcc -v mfloat.c
COLLECT_GCC_OPTIONS=’-v’ ‘-march=armv7-a’ ‘-mfloat-abi=hard’ ‘-mfpu=vfpv3-d16′ ‘-mthumb’
-mfloat-abi=hard

可看出使用hard硬件浮点模式。

**2、使用 arm-linux-gnueabi-gcc 编译：**
\# arm-linux-gnueabi-gcc -v mfloat.c
COLLECT_GCC_OPTIONS=’-v’ ‘-march=armv7-a’ ‘-mfloat-abi=softfp’ ‘-mfpu=vfpv3-d16′ ‘-mthumb’
-mfloat-abi=softfp

可看出使用softfp模式。