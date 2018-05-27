---
title: 'golang内存模型和happens_before'
date: 2018-05-19 11:45:58
tags:
- golang
---



### happens-before 术语

happens-before是一个术语，并不仅仅是Go语言才有的。简单的说，通常的定义如下：

假设A和B表示一个多线程的程序执行的两个操作。如果A happens-before B，那么A操作对内存的影响 将对执行B的线程(且执行B之前)可见。 



+ 无论使用哪种编程语言，有一点是相同的：如果操作A和B在相同的线程中执行，并且A操作的声明在B之前，那么A happens-before B。

```
int A, B;
void foo()
{
  // This store to A ...
  A = 5;
  // ... effectively becomes visible before the following loads. Duh!
  B = A * A;
}
```

 

+ 还有一点是，在每门语言中，无论你使用那种方式获得，happens-before关系都是可传递的：如果A happens-before B，同时B happens-before C，那么A happens-before C。当这些关系发生在不同的线程中，传递性将变得非常有用。

<!-- more -->



刚接触这个术语的人总是容易误解，这里必须澄清的是，happens-before并不是指时序关系，并不是说A happens-before B就表示操作A在操作B之前发生。它就是一个术语，就像光年不是时间单位一样。具体地说：

1.  **A happens-before B并不意味着A在B之前发生。**
2.  **A在B之前发生并不意味着A happens-before B。**

这两个陈述看似矛盾，其实并不是。如果你觉得很困惑，可以多读几篇它的定义。后面我会试着解释这点。记住，happens-before 是一系列语言规范中定义的操作间的关系。它和时间的概念独立。这和我们通常说”A在B之前发生”时表达的真实世界中事件的时间顺序不同。



### A happens-before B并不意味着A在B之前发生 (编译器可能会重排)



这里有个例子，其中的操作具有happens-before关系，但是实际上并不一定是按照那个顺序发生的。下面的代码执行了(1)对A的赋值，紧接着是(2)对B的赋值。

```
int A = 0;
int B = 0;
void main()
{
    A = B + 1; // (1)
    B = 1; // (2)
}
```



根据前面说明的规则，(1) happens-before (2)。但是，如果我们使用gcc -O2编译这个代码，编译器将产生一些指令重排序。有可能执行顺序是这样子的：

```
将B的值取到寄存器
将B赋值为1
将寄存器值加1后赋值给A
```

也就是到第二条机器指令(对B的赋值)完成时，对A的赋值还没有完成。换句话说，(1)并没有在(2)之前发生!

那么，这里违反了happens-before关系了吗？让我们来分析下，根据定义，操作(1)对内存的影响必须在操作(2)执行之前对其可见。换句话说，对A的赋值必须有机会对B的赋值有影响.

但是在这个例子中，对A的赋值其实并没有对B的赋值有影响。即便(1)的影响真的可见，(2)的行为还是一样。所以，这并不能算是违背happens-before规则。



### A在B之前发生并不意味着A happens-before B (虽然在之前发生但不满足规则)

下面这个例子中，所有的操作按照指定的顺序发生，但是并能不构成happens-before 关系。假设一个线程调用pulishMessage，同时，另一个线程调用consumeMessage。 由于我们并行的操作共享变量，为了简单，我们假设所有对int类型的变量的操作都是原子的。



```
int isReady = 0;
int answer = 0;
void publishMessage()
{
  answer = 42; // (1)
  isReady = 1; // (2)
}
void consumeMessage()
{
  if (isReady)			    // (3) <-- Let's suppose this line reads 1
  	printf("%d\n", answer); // (4)
}
```



根据程序的顺序，在(1)和(2)之间存在happens-before 关系，同时在(3)和(4)之间也存在happens-before关系。



除此之外，我们假设在运行时，isReady读到1(是由另一个线程在(2)中赋的值)。在这中情形下，我们可知(2)一定在(3)之前发生。但是这并不意味着在(2)和(3)之间存在happens-before 关系!

happens-before 关系只在语言标准中定义的地方存在，这里并没有相关的规则说明(2)和(3)之间存在happens-before关系，即便(3)读到了(2)赋的值。

还有，由于(2)和(3)之间，(1)和(4)之间都不存在happens-before关系，那么(1)和(4)的内存交互也可能被重排序 (要不然来自编译器的指令重排序，要不然来自处理器自身的内存重排序)。那样的话，即使(3)读到1，(4)也会打印出“0“。

 

### Go关于同步的规则 (往冰箱放西瓜, 先放后拿,往手里递西瓜, 先接后放)



关于channel的happens-before在Go的内存模型中提到了三种情况：

- 对一个channel的发送操作 happens-before 相应channel的接收操作完成     	  **(往冰箱放西瓜, 先放后拿)**
- 关闭一个channel happens-before 从该Channel接收到最后的返回值0                

- 不带缓冲的channel的接收操作 happens-before 相应channel的发送操作完成      **(往手里递西瓜, 先接后放) **     



先看一个简单的例子：

```
var c = make(chan int, 10)
var a string
func f() {
    a = "hello, world"  // (1)
    c <- 0  // (2)
}
func main() {
    go f()
    <-c   // (3)
    print(a)  // (4)
}
```

上述代码可以确保输出"hello, world"，因为(1) happens-before (2)，(4) happens-after (3)，再根据上面的第一条规则(2)是 happens-before (3)的，最后根据happens-before的可传递性，于是有(1) happens-before (4)，也就是a = "hello, world" happens-before print(a)。



再看另一个例子：

```
var c = make(chan int)
var a string
func f() {
    a = "hello, world"  // (1)
    <-c   // (2)
}
func main() {
    go f()
    c <- 0  // (3)
    print(a)  // (4)
}
```

根据上面的第三条规则(2) happens-before (3)，最终可以保证(1) happens-before (4)。





如果我把上面的代码稍微改一点点，将c变为一个带缓存的channel，则print(a)打印的结果不能够保证是"hello world"。

```
var c = make(chan int, 1)
var a string
func f() {
    a = "hello, world"  // (1)
    <-c   // (2)
}
func main() {
    go f()
    c <- 0  // (3)
    print(a)  // (4)
}
```

因为这里不再有任何同步保证，使得(2) happens-before (3)。可以回头分析一下本节最前面的例子，也是没有保证happens-before条件。





### golang happen before 的保证



**1) 单线程**



**2) Init 函数**

- 如果包P1中导入了包P2，则P2中的init函数Happens Before 所有P1中的操作
- main函数Happens After 所有的init函数



3) **Goroutine**

- Goroutine的创建Happens Before所有此Goroutine中的操作
- Goroutine的销毁Happens After所有此Goroutine中的操作



 **4) Channel**

- 对一个元素的send操作Happens Before对应的receive 完成操作
- 对channel的close操作Happens Before receive 端的收到关闭通知操作
- 对于Unbuffered Channel，对一个元素的receive 操作Happens Before对应的send完成操作
- 对于Buffered Channel，假设Channel 的buffer 大小为C，那么对第k个元素的receive操作，Happens Before第k+C个send完成操作。可以看出上一条Unbuffered Channel规则就是这条规则C=0时的特例



**5) Lock**

Go里面有Mutex和RWMutex两种锁，RWMutex除了支持互斥的Lock/Unlock，还支持共享的RLock/RUnlock。

- 对于一个Mutex/RWMutex，设n < m，则第n个Unlock操作Happens Before第m个Lock操作。
- 对于一个RWMutex，存在数值n，RLock操作Happens After 第n个UnLock，其对应的RUnLock Happens Before 第n+1个Lock操作。

*简单理解就是这一次的Lock总是Happens After上一次的Unlock，读写锁的RLock HappensAfter上一次的UnLock，其对应的RUnlock Happens Before 下一次的Lock。*

```
var l sync.Mutex
var a string
func f() {
    a = "hello, world" // (1)
    l.Unlock() // (2)
}
func main() {
    l.Lock() // (3)
    go f()
    l.Lock() // (4)
    print(a) // (5)
}
```

(1) happens-before (2) happens-before (4) happens-before (5)



**6) Once**

once.Do中执行的操作，Happens Before 任何一个once.Do调用的返回。