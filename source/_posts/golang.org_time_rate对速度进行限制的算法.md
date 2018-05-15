
---
title: 'golang.org/x/time/rate对速度进行限制的算法'
date: 2018-05-14 22:47:26
tags:
- golang
---


### Limiter

```
func NewLimiter(r Limit, b int) *Limiter {
	return &Limiter{
		limit: r,
		burst: b,
	}
}
```
Limter限制时间的发生频率，采用令牌池的算法实现。这个池子一开始容量为b，装满b个令牌，然后每秒往里面填充r个令牌。 

由于令牌池中最多有b个令牌，所以一次最多只能允许b个事件发生，一个事件花费掉一个令牌。


```
l := rate.NewLimiter(1, 3) 
//第一个参数为每秒发生多少次事件，第二个参数是最大可运行多少个事件
```


<!-- more -->

### Wait/WaitN 当没有可用事件时，将阻塞等待

```
func main() {
	l := rate.NewLimiter(1, 3)

	c, _ := context.WithCancel(context.TODO())
	for {
		l.Wait(c)
		fmt.Println(time.Now().Format("04:05.000"))
	}
}

39:58.446
39:58.446
39:58.446
39:59.451
40:00.451
40:01.450
40:02.450
```

### Allow/AllowN 当没有可用事件时，返回false


```
func main() {
	l := rate.NewLimiter(1, 3)

	for {
		if l.AllowN(time.Now(), 1) {
			fmt.Println(time.Now().Format("04:05.000"))
		} else {
			time.Sleep(1 * time.Second / 10)
			fmt.Println(time.Now().Format("Second 04:05.000"))
		}
	}
}


43:32.534
43:32.534
43:32.534
Second 43:32.635
Second 43:32.737
Second 43:32.838
Second 43:32.938
Second 43:33.039
Second 43:33.144
Second 43:33.249
Second 43:33.350
Second 43:33.451
Second 43:33.552
43:33.552
Second 43:33.653
Second 43:33.753
Second 43:33.854
Second 43:33.955
Second 43:34.055
```


### Reserve/ReserveN 当没有可用事件时，返回 Reservation，和要等待多久才能获得足够的事件


```
func main() {
	l := rate.NewLimiter(1, 3)

	for {
		r := l.ReserveN(time.Now(), 1)
		s := r.Delay()
		time.Sleep(s)
		fmt.Println(s, time.Now().Format("04:05.000"))
	}
}


0s 44:54.118
0s 44:54.119
0s 44:54.119
999.857594ms 44:55.124
994.670516ms 44:56.124
994.778299ms 44:57.124
994.763486ms 44:58.124
```
