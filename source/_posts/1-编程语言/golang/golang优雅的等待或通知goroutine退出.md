---
title: golang优雅的等待或通知goroutine退出
tags: ["golang"]
abbrlink: aca47db0
categories:
  - 1-编程语言
  - golang
date: 2018-05-28 00:03:21
---



### 优雅的等待goroutine退出



#### 通过Channel传递退出信号

Go的一大设计哲学就是：通过Channel共享数据，而不是通过共享内存共享数据。主流程可以通过channel向任何goroutine发送停止信号，就像下面这样：



这种方式可以实现优雅地停止goroutine，但是当goroutine特别多的时候，这种方式不管在代码美观上还是管理上都显得笨拙不堪。

```
package main

import (
   "fmt"
   "time"
)

func run(done chan int) {
   for {
      select {
      case <-done:
         fmt.Println("exiting...")
         done <- 1
         break
      default:
      }

      time.Sleep(time.Second * 1)
      fmt.Println("do something")
   }
}

func main() {
   c := make(chan int)

   go run(c)

   fmt.Println("wait")
   time.Sleep(time.Second * 5)

   c <- 1
   <-c

   fmt.Println("main exited")
}
```

<!-- more -->

#### 使用Waitgroup

通常情况下，我们像下面这样使用waitgroup:

1. 创建一个Waitgroup的实例，假设此处我们叫它wg
2. 在每个goroutine启动的时候，调用wg.Add(1)，这个操作可以在goroutine启动之前调用，也可以在goroutine里面调用。当然，也可以在创建n个goroutine前调用wg.Add(n)
3. 当每个goroutine完成任务后，调用wg.Done()
4. 在等待所有goroutine的地方调用wg.Wait()，它在所有执行了wg.Add(1)的goroutine都调用完wg.Done()前阻塞，当所有goroutine都调用完wg.Done()之后它会返回。

那么，如果我们的goroutine是一匹不知疲倦的牛，一直孜孜不倦地工作的话，如何在主流程中告知并等待它退出呢？像下面这样做：



```
package main

import (
   "fmt"
   "os"
   "os/signal"
   "sync"
   "syscall"
)

type Service struct {
   // Other things

   ch        chan bool
   waitGroup *sync.WaitGroup
}

func NewService() *Service {
   s := &Service{
      // Init Other things
      ch:        make(chan bool),
      waitGroup: &sync.WaitGroup{},
   }

   return s
}

func (s *Service) Stop() {
   close(s.ch)
   s.waitGroup.Wait()
}

func (s *Service) Serve() {
   s.waitGroup.Add(1)
   defer s.waitGroup.Done()

   for {
      select {
      case <-s.ch:
         fmt.Println("stopping...")
         return
      default:
      }
      s.waitGroup.Add(1)
      go s.anotherServer()
   }
}
func (s *Service) anotherServer() {
   defer s.waitGroup.Done()
   for {
      select {
      case <-s.ch:
         fmt.Println("stopping...")
         return
      default:
      }

      // Do something
   }
}

func main() {

   service := NewService()
   go service.Serve()

   // Handle SIGINT and SIGTERM.
   ch := make(chan os.Signal)
   signal.Notify(ch, syscall.SIGINT, syscall.SIGTERM)
   fmt.Println(<-ch)

   // Stop the service gracefully.
   service.Stop()
}
```



### 优雅的通知 goroutine 退出



有时候我们需要通知goroutine停止它正在干的事情，比如一个正在执行计算的web服务，然而它的客户端已经断开了和服务端的连接。

Go语言并没有提供在一个goroutine中终止另一个goroutine的方法，由于这样会导致goroutine之间的共享变量落在未定义的状态上。

在rocket launch程序中，我们往名字叫abort的channel里发送了一个简单的值，在countdown的goroutine中会把这个值理解为自己的退出信号。但是如果我们想要退出两个或者任意多个goroutine怎么办呢？



一种可能的手段是向abort的channel里发送和goroutine数目一样多的事件来退出它们。如果这些goroutine中已经有一些自己退出了，那么会导致我们的channel里的事件数比goroutine还多，这样导致我们的发送直接被阻塞。另一方面，如果这些goroutine又生成了其它的goroutine，我们的channel里的数目又太少了，所以有些goroutine可能会无法接收到退出消息。一般情况下我们是很难知道在某一个时刻具体有多少个goroutine在运行着的。



另外，当一个goroutine从abort channel中接收到一个值的时候，他会消费掉这个值，这样其它的goroutine就没法看到这条信息。为了能够达到我们退出goroutine的目的，我们需要更靠谱的策略，来通过一个channel把消息广播出去，这样goroutine们能够看到这条事件消息，并且在事件完成之后，可以知道这件事已经发生过了。



回忆一下我们关闭了一个channel并且被消费掉了所有已发送的值，操作channel之后的代码可以立即被执行，并且会产生零值。我们可以将这个机制扩展一下，来作为我们的广播机制：***不要向channel发送值，而是用关闭一个channel来进行广播。***





### 优雅的控制 goroutine 退出

通常`Goroutine`会因为两种情况阻塞：

1. IO操作，比如对`Socket`的`Read`。
2. `channel`操作。对一个chan的读写都有可能阻塞`Goroutine`。



对于情况1，只需要关闭对应的描述符，阻塞的`Goroutine`自然会被唤醒。

重点讨论情况2。并发编程，`Goroutine`提供一种`channel`机制，`channel`类似管道，写入者向里面写入数据，读取者从中读取数据。如果`channel`里面没有数据，读取者将阻塞，直到有数据；如果`channel`里面数据满了，写入者将因为无法继续写入数据而阻塞。

如果在整个应用程序的生命周期里，writer和reader都表现为一个`Goroutine`，始终都在工作，那么如何在应用程序结束前，通知它们终止呢？在Go中，并不推荐像abort线程那样，强行的终止`Goroutine`。因此，抽象的说，必然需要保留一个入口，能够跟writer或reader通信，以告知它们终止。

 



我们先看reader。我们首先可以想到，利用`close`函数关闭正在读取的`channel`，从而可以唤醒reader，并退出。但是考虑到`close`并不能很好的处理writer（因为writer试图写入一个已经close的channel，将引发异常）。因此，我们需要设计一个额外的只读`channel`用于通知：

```
type routineSignal struct {
    done <-chan struct{}
}
```

 `routineSignal`的实例，应当通过外部生成并传递给reader，例如：

```
func (r *reader)init(s *routineSignal) {
    r.signal = s
}
```

 在reader的循环中，就可以这么写：

```
func (r *reader)loop() {
    for {
        select {
        case <-r.signal.done:
            return
        case <-r.queue:
            ....
        }
    }
}
```

当需要终止`Goroutine`的时候只需要关闭这个额外的`channel`：

```
close(signal.done)
```

 



看起来很完备了，这可以处理大部分的情况了。这样做有个弊端，尽管，我们可以期望`close`唤醒`Goroutine`进而退出，但是并不能知道`Goroutine`什么时候完成退出，因为`Goroutine`可能在退出前还有一些善后工作，这个时候我们需要`sync.WaitGroup`。改造一下`routineSignal`：



```
type routineSignal struct {
    done chan struct{}
    wg   sync.WaitGroup
}
```



增加一个sync.WaitGroup的实例，在`Goroutine`开始工作时，对wg加1，在`Goroutine`退出前，对wg减1：

```
func (r *reader)loop() {
    r.signal.wg.Add(1)
    defer r.signal.wg.Done()
    for {
        select {
        case <-r.signal.done:
            return
        case <-r.queue:
            ....
        }
    }
}
```

 外部，只需要等待`WaitGroup`返回即可：

```
close(signal.done)
signal.wg.Wait()
```

只要`Wait()`返回就能断定`Goroutine`结束了。



推导一下，不难发现，对于writer也可以采用这种方法。于是，总结一下，我们创建了一个叫`routineSignal`的结构，结构里面包含一个`chan`用来通知`Goroutine`结束，包含一个`WaitGroup`用于`Goroutine`通知外部完成善后。这样，通过这个结构的实例优雅的终止`Goroutine`，而且还可以确保`Goroutine`终止成功。 