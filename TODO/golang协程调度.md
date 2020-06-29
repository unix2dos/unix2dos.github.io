---
title: "golang协程调度"
date: 2020-01-02 00:00:00
tags:
- golang
---



# 1. 基础术语

### 1.1 并发

一个cpu上能同时执行多项任务，在很短时间内，cpu来回切换任务执行(在某段很短时间内执行程序a，然后又迅速得切换到程序b去执行)，有时间上的重叠（宏观上是同时的，微观仍是顺序执行）,这样看起来多个任务像是同时执行，这就是并发。

<!-- more -->

### 1.2 并行

当系统有多个CPU时,每个CPU同一时刻都运行任务，互不抢占自己所在的CPU资源，同时进行，称为并行。

### 1.3 进程

cpu在切换程序的时候，如果不保存上一个程序的状态（也就是我们常说的context--上下文），直接切换下一个程序，就会丢失上一个程序的一系列状态，于是引入了进程这个概念，用以划分好程序运行时所需要的资源。因此进程就是一个程序运行时候的所需要的基本资源单位（也可以说是程序运行的一个实体）。

### 1.4 线程

cpu切换多个进程的时候，会花费不少的时间，因为切换进程需要切换到内核态，而每次调度需要内核态都需要读取用户态的数据，进程一旦多起来，cpu调度会消耗一大堆资源，因此引入了线程的概念，线程本身几乎不占有资源，他们共享进程里的资源，内核调度起来不会那么像进程切换那么耗费资源。

### 1.5 协程

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此，协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作执行者则是用户自身程序，goroutine也是协程。



# 2. golang 调度模型

### 2.1 调度器的三个基本对象

![1](golang协程调度/4.png)

- M (Work Thread)，M代表内核级线程，一个M就是一个线程，goroutine就是跑在M之上的；M是一个很大的结构，里面维护小对象内存cache（mcache）、当前执行的goroutine、随机数发生器等等非常多的信息
- G (Goroutine)，代表一个goroutine，它有自己的栈，instruction pointer和其他信息（正在等待的channel等等），用于调度。
- P (Processor)，处理器，它的主要用途就是用来执行goroutine的，所以它也维护了一个goroutine队列，里面存储了所有需要它来执行的goroutine   



### 2.2 调度实现

![1](golang协程调度/1.png)

从上图中看，有2个物理线程M，每一个M都拥有一个处理器P，每一个也都有一个正在运行的goroutine。

+ P的数量可以通过GOMAXPROCS() 来设置，它其实也就代表了真正的并发度，即有多少个goroutine可以同时运行。
+ 图中灰色的那些goroutine并没有运行，而是出于ready的就绪态，正在等待被调度。P维护着这个队列（称之为runqueue）
+ Go语言里，启动一个goroutine很容易：go function 就行，所以每有一个go语句被执行，runqueue队列就在其末尾加入一个

### 2.3 线程阻塞

当一个OS线程M0陷入阻塞时（如下图)， 例如协程G0遇到阻塞调用（比如系统调用syscall）超过10ms时 ，这个G0连同M0 会被剥离出P，产生一个新的M1 来接手P(或者从线程缓存中取出)

![1](golang协程调度/2.png)



当MO返回时，它必须尝试取得一个P来运行goroutine，一般情况下，它会从其他的OS线程那里拿一个P过来，如果没有拿到的话，它就把goroutine放在一个global runqueue里，然后自己睡眠（放入线程缓存里）。

所有的P也会周期性的检查global runqueue并运行其中的goroutine，否则global runqueue上的goroutine永远无法执行。 

### 2.4 工作量窃取

P所分配的任务G很快就执行完了（分配不均），这就导致了这个处理器P很忙，但是其他的P还有任务，此时如果global runqueue没有任务G了，那么P不得不从其他的P里拿一些G来执行。

一般来说，如果P从其他的P那里要拿任务的话，一般就拿run queue的一半，这就确保了每个OS线程都能充分的使用，如下图：

![1](golang协程调度/3.png)



# 3. 使用







# 5. 参考资料



+ https://studygolang.com/articles/26795 //TODO:
+ https://www.pengrl.com/p/29953/ //TODO:

+ https://my.oschina.net/linker/blog/1504199  //TODO:



[https://txiner.top/post/%E4%B8%80%E9%81%93%E9%97%AE%E9%A2%98%E5%BC%95%E5%8F%91%E7%9A%84golang%E8%B0%83%E5%BA%A6/](https://txiner.top/post/一道问题引发的golang调度/)

https://www.cnblogs.com/sunsky303/p/9705727.html

https://blog.csdn.net/weixin_34191845/article/details/91818888



https://www.cnblogs.com/zkweb/p/7815600.html

https://segmentfault.com/a/1190000015464889



https://www.cnblogs.com/secondtonone1/p/11803961.html