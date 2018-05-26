happens-before
happens-before是一个术语，并不仅仅是Go语言才有的。简单的说，通常的定义如下：
假设A和B表示一个多线程的程序执行的两个操作。如果A happens-before B，那么A操作对内存的影响 将对执行B的线程(且执行B之前)可见。


无论使用哪种编程语言，有一点是相同的：如果操作A和B在相同的线程中执行，并且A操作的声明在B之前，那么A happens-before B。
int A, B;
void foo()
{
  // This store to A ...
  A = 5;
  // ... effectively becomes visible before the following loads. Duh!
  B = A * A;
}
还有一点是，在每门语言中，无论你使用那种方式获得，happens-before关系都是可传递的：如果A happens-before B，同时B happens-before C，那么A happens-before C。当这些关系发生在不同的线程中，传递性将变得非常有用。
刚接触这个术语的人总是容易误解，这里必须澄清的是，happens-before并不是指时序关系，并不是说A happens-before B就表示操作A在操作B之前发生。它就是一个术语，就像光年不是时间单位一样。具体地说：
  1. A happens-before B并不意味着A在B之前发生。
  2. A在B之前发生并不意味着A happens-before B。

这两个陈述看似矛盾，其实并不是。如果你觉得很困惑，可以多读几篇它的定义。后面我会试着解释这点。记住，happens-before 是一系列语言规范中定义的操作间的关系。它和时间的概念独立。这和我们通常说”A在B之前发生”时表达的真实世界中事件的时间顺序不同。



A happens-before B并不意味着A在B之前发生
这里有个例子，其中的操作具有happens-before关系，但是实际上并不一定是按照那个顺序发生的。下面的代码执行了(1)对A的赋值，紧接着是(2)对B的赋值。
int A = 0;
int B = 0;
void main()
{
    A = B + 1; // (1)
    B = 1; // (2)
}
根据前面说明的规则，(1) happens-before (2)。但是，如果我们使用gcc -O2编译这个代码，编译器将产生一些指令重排序。有可能执行顺序是这样子的：
将B的值取到寄存器
将B赋值为1
将寄存器值加1后赋值给A
也就是到第二条机器指令(对B的赋值)完成时，对A的赋值还没有完成。
换句话说，(1)并没有在(2)之前发生!


那么，这里违反了happens-before关系了吗？让我们来分析下，根据定义，操作(1)对内存的影响必须在操作(2)执行之前对其可见。换句话说，对A的赋值必须有机会对B的赋值有影响.
但是在这个例子中，对A的赋值其实并没有对B的赋值有影响。即便(1)的影响真的可见，(2)的行为还是一样。所以，这并不能算是违背happens-before规则。




A在B之前发生并不意味着A happens-before B
下面这个例子中，所有的操作按照指定的顺序发生，但是并能不构成happens-before 关系。假设一个线程调用pulishMessage，同时，另一个线程调用consumeMessage。 由于我们并行的操作共享变量，为了简单，我们假设所有对int类型的变量的操作都是原子的。
int isReady = 0;
int answer = 0;
void publishMessage()
{
  answer = 42; // (1)
  isReady = 1; // (2)
}
void consumeMessage()
{
  if (isReady) // (3) <-- Let's suppose this line reads 1
  printf("%d\n", answer); // (4)
}
根据程序的顺序，在(1)和(2)之间存在happens-before 关系，同时在(3)和(4)之间也存在happens-before关系。
除此之外，我们假设在运行时，isReady读到1(是由另一个线程在(2)中赋的值)。在这中情形下，我们可知(2)一定在(3)之前发生。但是这并不意味着在(2)和(3)之间存在happens-before 关系!
happens-before 关系只在语言标准中定义的地方存在，这里并没有相关的规则说明(2)和(3)之间存在happens-before关系，即便(3)读到了(2)赋的值。
还有，由于(2)和(3)之间，(1)和(4)之间都不存在happens-before关系，那么(1)和(4)的内存交互也可能被重排序 (要不然来自编译器的指令重排序，要不然来自处理器自身的内存重排序)。那样的话，即使(3)读到1，(4)也会打印出“0“。




Go关于同步的规则
我们回过头来再看看"The Go Memory Model"中关于happens-before的部分。
如果满足下面条件，对变量v的读操作r可以侦测到对变量v的写操作w：
  1. r does not happen before w.
  2. There is no other write w to v that happens after w but before r.
为了保证对变量v的读操作r可以侦测到某个对v的写操作w，必须确保w是r可以侦测到的唯一的写操作。也就是说当满足下面条件时可以保证读操作r能侦测到写操作w：
  1. w happens-before r.
  2. Any other write to the shared variable v either happens-before w or after r.
关于channel的happens-before在Go的内存模型中提到了三种情况：
  1. 对一个channel的发送操作 happens-before 相应channel的接收操作完成 //先发送才能接
  2. 关闭一个channel happens-before 从该Channel接收到最后的返回值0  //先关闭才能0
  3. 不带缓冲的channel的接收操作 happens-before 相应channel的发送操作完成//不带缓冲先接

先看一个简单的例子：
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
上述代码可以确保输出"hello, world"，因为(1) happens-before (2)，(4) happens-after (3)，再根据上面的第一条规则(2)是 happens-before (3)的，最后根据happens-before的可传递性，于是有(1) happens-before (4)，也就是a = "hello, world" happens-before print(a)。
再看另一个例子：
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
根据上面的第三条规则(2) happens-before (3)，最终可以保证(1) happens-before (4)。
如果我把上面的代码稍微改一点点，将c变为一个带缓存的channel，则print(a)打印的结果不能够保证是"hello world"。
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
因为这里不再有任何同步保证，使得(2) happens-before (3)。可以回头分析一下本节最前面的例子，也是没有保证happens-before条件。






同步方法
初始化
  1. 如果package p 引用了package q，q的init()方法 happens-before p （Java工程师可以对比一下final变量的happens-before规则）
  2. main.main()方法 happens-after所有package的init()方法结束。
创建Goroutine
  1. go语句创建新的goroutine happens-before 该goroutine执行（这个应该很容易理解）
package main

import (
	"log"
	"time"
)

var a, b, c int

func main() {
	a = 1
	b = 2
	go func() {
		c = a + 2
		log.Println(a, b, c)
	}()
	time.Sleep(1 * time.Second)
}

利用这条happens-before，我们可以确定c=a+2是happens-aftera=1和b=2，所以结果输出是可以确定的1 2 3，但如果是下面这样的代码，输出就不确定了，有可能是1 2 3或0 0 2
func main() {
	go func() {
		c = a + 2
		log.Println(a, b, c)
	}()
	a = 1
	b = 2
	time.Sleep(1 * time.Second)
}

销毁Goroutine
  1. Goroutine的退出并不保证happens-before任何事件。
var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}

上面代码因为a="hello" 没有使用同步事件，并不能保证这个赋值被主goroutine可见。事实上，极度优化的Go编译器甚至可以完全删除这行代码go func() { a = "hello" }()。
Goroutine对变量的修改需要让对其它Goroutine可见，除了使用锁来同步外还可以用Channel。
Channel通信
在Go编程中，Channel是被推荐的执行体间通信的方法，Go的编译器和运行态都会尽力对其优化。
  1. 对一个Channel的发送操作(send) happens-before 相应Channel的接收操作完成
  2. 关闭一个Channel happens-before 从该Channel接收到最后的返回值0
  3. 不带缓冲的Channel的接收操作（receive） happens-before 相应Channel的发送操作完成
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}

上述代码可以确保输出hello, world，因为a = "hello, world" happens-before c <- 0，print(a) happens-after <-c， 根据上面的规则1）以及happens-before的可传递性，a = "hello, world" happens-beforeprint(a)。
根据规则2）把c<-0替换成close(c)也能保证输出hello,world，因为关闭操作在<-c接收到0之前发送。
var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}

根据规则3），因为c是不带缓冲的Channel，a = "hello, world" happens-before <-c happens-before c <- 0happens-before print(a)， 但如果c是缓冲队列，如定义c = make(chan int, 1), 那结果就不确定了。
锁
sync 包实现了两种锁数据结构:
  1. sync.Mutex -> java.util.concurrent.ReentrantLock
  2. sync.RWMutex -> java.util.concurrent.locks.ReadWriteLock
其happens-before规则和Java的也类似：
  1. 任何sync.Mutex或sync.RWMutex 变量（l），定义 n < m， 第n次 l.Unlock() happens-before 第m次l.lock()调用返回。
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}

a = "hello, world" happens-before l.Unlock() happens-before 第二个 l.Lock() happens-before print(a)





