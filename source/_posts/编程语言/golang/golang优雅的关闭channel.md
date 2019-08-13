---
title: golang优雅的关闭channel
tags:
  - golang
abbrlink: 8b210700
categories:
  - 编程语言
  - golang
date: 2018-05-27 23:31:19
---



### Channel使用规范

在不能更改channel状态的情况下，没有简单普遍的方式来检查channel是否已经关闭了

关闭已经关闭的channel会导致panic，所以在closer(关闭者)不知道channel是否已经关闭的情况下去关闭channel是很危险的

发送值到已经关闭的channel会导致panic，所以如果sender(发送者)在不知道channel是否已经关闭的情况下去向channel发送值是很危险的

 

### The Channel Closing Principle

在使用Go channel的时候，一个适用的原则是*不要从接收端关闭channel，也不要关闭有多个并发发送者的channel*。

换句话说，如果sender(发送者)只是唯一的sender或者是channel最后一个活跃的sender，那么你应该在sender的goroutine关闭channel，从而通知receiver(s)(接收者们)已经没有值可以读了。维持这条原则将保证永远不会发生向一个已经关闭的channel发送值或者关闭一个已经关闭的channel。

<!-- more -->

### 打破Channel Closing Principle的解决方案 

 

如果你因为某种原因从接收端（receiver side）关闭channel或者在多个发送者中的一个关闭channel，那么你应该使用列在[Golang panic/recover Use Cases](https://link.jianshu.com/?t=http://www.tapirgames.com/blog/golang-panic-use-cases)的函数来安全地发送值到channel中（假设channel的元素类型是T）

 

```
func SafeSend(ch chan T, value T) (closed bool) {
    defer func() {
        if recover() != nil {
            // the return result can be altered 
            // in a defer function call
            closed = true
        }
    }()
    
    ch <- value // panic if ch is closed
    return false // <=> closed = false; return
}
```

 

同样的想法也可以用在从多个goroutine关闭channel中：



```
func SafeClose(ch chan T) (justClosed bool) {
	defer func() {
		if recover() != nil {
			justClosed = false
		}
	}()
	
	// assume ch != nil here.
	close(ch) // panic if ch is closed
	return true // <=> justClosed = true; return
}
```



很多人喜欢用`sync.Once`来关闭channel：

```
 type MyChannel struct {
	C    chan T
	once sync.Once
}

func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan T)}
}

func (mc *MyChannel) SafeClose() {
	mc.once.Do(func() {
		close(mc.C)
	})
}
```



要知道golang的设计者不提供SafeClose或者SafeSend方法是有原因的，他们本来就不推荐在消费端或者在并发的多个生产端关闭channel，比如关闭只读channel在语法上就彻底被禁止使用了。

 

### 优雅的关闭Channel的方法

上文的SafeSend方法一个很大的劣势在于它不能用在select块的case语句中。而另一个很重要的劣势在于像我这样对代码有洁癖的人来说，使用panic/recover和sync/mutex来搞定不是那么的优雅。下面我们引入在不同的场景下可以使用的纯粹的优雅的解决方法。

 

#### 多个消费者，单个生产者。

这种情况最简单，直接让生产者关闭channel好了。 



```
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)
	

	const MaxRandomNumber = 100000
	const NumReceivers = 100
	
	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)
	

	dataCh := make(chan int, 100)
	
	// 一个生产者
	go func() {
		for {
			if value := rand.Intn(MaxRandomNumber); value == 0 {
				close(dataCh)
				return
			} else {			
				dataCh <- value
			}
		}
	}()
	
	// 多个消费者
	for i := 0; i < NumReceivers; i++ {
		go func() {
			defer wgReceivers.Done()
			
			for value := range dataCh {
				log.Println(value)
			}
		}()
	}
	
	wgReceivers.Wait()
}
```



#### 多个生产者，单个消费者。

这种情况要比上面的复杂一点。我们不能在消费端关闭channel，因为这违背了channel关闭原则。但是我们可以让消费端关闭一个附加的信号来通知发送端停止生产数据。



```
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)
	

	const MaxRandomNumber = 100000
	const NumSenders = 1000
	
	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(1)
	
	
	dataCh := make(chan int, 100)
	stopCh := make(chan struct{})
	
	
	// 多个生产者
	for i := 0; i < NumSenders; i++ {
		go func() {
			for {
		
				// 目的是尝试退出, 因为越早越好, 此处可以省略, 因为就算多发送了值, 消费者也不会理会了
				select {
				case <- stopCh:
					return
				default:
				}
			

				select {
				case <- stopCh:
					return
				case dataCh <- rand.Intn(MaxRandomNumber):
				}
			}
		}()
	}
	
	// 一个消费者
	go func() {
		defer wgReceivers.Done()
		
		for value := range dataCh {
			if value == MaxRandomNumber-1 {
				
				// 这里即是dataCh 的消费者, 也是 stopCh 的生产者
				close(stopCh)
				return
			}
			
			log.Println(value)
		}
	}()
	
	
	wgReceivers.Wait()
}
```



就上面这个例子，生产者同时也是退出信号channel的接受者，退出信号channel仍然是由它的生产端关闭的，所以这仍然没有违背**channel关闭原则**。值得注意的是，这个例子中生产端和接受端都没有关闭消息数据的channel，channel在没有任何goroutine引用的时候会自行关闭，而不需要显示进行关闭。

 

####  多个生产者，多个消费者

 

这是最复杂的一种情况，我们既不能让接受端也不能让发送端关闭channel。我们甚至都不能让接受者关闭一个退出信号来通知生产者停止生产。因为我们不能违反**channel关闭原则**。但是我们可以引入一个额外的协调者来关闭附加的退出信号channel。  

 

```
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
	"strconv"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)
	

	const MaxRandomNumber = 100000
	const NumReceivers = 10
	const NumSenders = 1000
	
	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)
	
	
	dataCh := make(chan int, 100)
	stopCh := make(chan struct{}) //生产者是主持人, 消费者是 (dataCh所有生产者和消费者)
	
	toStop := make(chan string, 1) //作用是通知主持人去关闭stopCh, 生产者是 (dataCh所有生产者和消费者) 消费者是主持人
		
	
	var stoppedBy string
	
	// 主持人
	go func() {
		stoppedBy = <- toStop
		close(stopCh)
	}()
	
	// 多个生产者
	for i := 0; i < NumSenders; i++ {
		go func(id string) {
			for {
				value := rand.Intn(MaxRandomNumber)
				if value == 0 {
					//通知主持人去干关闭的活
					select {
					case toStop <- "sender#" + id:
					default:
					}
					return
				}
				
			
                //尝试尽早退出, 这里不能省略, 因为可能会导致多发送一次
				select {
				case <- stopCh:
					return
				default:
				}
				

				select {
				case <- stopCh:
					return
				case dataCh <- value:
				}
			}
		}(strconv.Itoa(i))
	}
	
	// 多个消费者
	for i := 0; i < NumReceivers; i++ {
		go func(id string) {
			defer wgReceivers.Done()
			
			for {
				//尝试尽早退出, 这里不能省略, 因为可能会导致多接收一次
				select {
				case <- stopCh:
					return
				default:
				}
				
			
				//注意此处如果 stopCh 关闭了, 下面也有能 return 不了
                //因为dataCh也有可能select 到, 所以上一个 select语句不能省略
                
                
				select {
				case <- stopCh:
					return
				case value := <-dataCh:
					if value == MaxRandomNumber-1 {
						//通知主持人去干关闭的活
						select {
						case toStop <- "receiver#" + id:
						default:
						}
						return
					}
					
					log.Println(value)
				}
			}
		}(strconv.Itoa(i))
	}
	
	
	wgReceivers.Wait()
	log.Println("stopped by", stoppedBy)
}
```



在这个例子中，仍然遵守着*channel closing principle*。 请注意channel `toStop`的缓冲大小是1.这是为了避免当mederator goroutine 准备好之前第一个通知就已经发送了，导致丢失。



#### **结论**

没有任何场景值得你去打破channel关闭原则，如果你遇到这样的一种特殊场景，还是建议你好好思考一下自己设计，是不是该重构一下了。





#### [个人疑问解答](https://www.jianshu.com/p/d24dfbb33781)

楼主你好, 关于第三个例子有些问题请教

1. value==0时, 为什么还要加个select, 不能直接发送给toStop吗?

```
if value == 0 {
select {
case toStop <- "sender#" + id:
default:
}
return
}
```
> 因为可能多个生产者或者多个消费者满足条件, 防止阻塞



2. select stopCh 为什么写了两次? 第一个select可以省略吗?

```
select {
case <- stopCh:
	return
default:
}

select {
case <- stopCh:
	return
case dataCh <- value:
}
```
> 为了尽早退出, 因为第二个 Select有可能 select 到dataCh, 虽然已经通知关闭了



3. toStop的缓冲大小是1, 为了避免准备好之前通知就发送了怎么理解??

   请注意channel toStop的缓冲大小是1.这是为了避免当mederator goroutine 准备好之前第一个通知就已经发送了，导致丢失。

> 因为有缓冲的 发送 happens_before 接收之前, 所以mederator能保证接收到数据
>
> 无缓冲的 接收 happens_before 发送之间,  可能会丢失数据