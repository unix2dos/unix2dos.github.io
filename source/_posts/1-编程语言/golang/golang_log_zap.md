---
title: golang的log库zap的使用
tags:
  - golang
categories:
  - 1-编程语言
  - golang
abbrlink: 5d303099
date: 2019-08-24 13:12:46
---



### 0. 前言

日志作为整个代码行为的记录，是程序执行逻辑和异常最直接的反馈。对于整个系统来说，日志是至关重要的组成部分。通过分析日志我们不仅可以发现系统的问题，同时日志中也蕴含了大量有价值可以被挖掘的信息，因此合理地记录日志是十分必要的。

<!-- more -->

### 1. golang log libs

目前golang主流的 log库有

+ https://github.com/uber-go/zap
+ https://github.com/Sirupsen/logrus

zap 跟 logrus 以及目前主流的 go 语言 log 类似，提倡采用结构化的日志格式，而不是将所有消息放到消息体中，简单来讲，日志有两个概念：字段和消息。字段用来结构化输出错误相关的上下文环境，而消息简明扼要的阐述错误本身。

##### 1.1 log库使用和性能对比

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"time"

	"github.com/golang/glog"
	"github.com/sirupsen/logrus"
	"go.uber.org/zap"
)

type dummy struct {
	Foo string `json:"foo"`
	Bar string `json:"bar"`
}

const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
const (
	letterIdxBits = 6                    // 6 bits to represent a letter index
	letterIdxMask = 1<<letterIdxBits - 1 // All 1-bits, as many as letterIdxBits
	letterIdxMax  = 63 / letterIdxBits   // # of letter indices fitting in 63 bits
)

func RandString(n int) string {
	b := make([]byte, n)
	// A rand.Int63() generates 63 random bits, enough for letterIdxMax letters!
	for i, cache, remain := n-1, rand.Int63(), letterIdxMax; i >= 0; {
		if remain == 0 {
			cache, remain = rand.Int63(), letterIdxMax
		}
		if idx := int(cache & letterIdxMask); idx < len(letterBytes) {
			b[i] = letterBytes[idx]
			i--
		}
		cache >>= letterIdxBits
		remain--
	}
	return string(b)
}

func dummyData() interface{} {
	return dummy{
		Foo: RandString(12),
		Bar: RandString(16),
	}
}

func main() {

	// logrus
	var x int64 = 0
	t := time.Now()
	for i := 0; i < 10000; i++ {
		logrus.WithField("Dummy", dummyData()).Infoln("this is a dummy log")
	}
	x += time.Since(t).Nanoseconds()

	// zap
	zlogger, _ := zap.NewProduction()
	sugar := zlogger.Sugar()
	var y int64 = 0
	t = time.Now()
	for i := 0; i < 10000; i++ {
		sugar.Infow("this is a dummy log", "Dummy", dummyData())
	}
	y += time.Since(t).Nanoseconds()

	// stdlog
	var z int64 = 0
	t = time.Now()
	for i := 0; i < 10000; i++ {
		dummyStr, _ := json.Marshal(dummyData())
		log.Printf("this is a dummy log: %s\n", string(dummyStr))
	}
	z += time.Since(t).Nanoseconds()

	// glog
	var w int64 = 0
	t = time.Now()
	for i := 0; i < 10000; i++ {
		glog.Info("\nthis is a dummy log: ", dummyData())
	}
	w += time.Since(t).Nanoseconds()

	// print
	fmt.Println("=====================")
	fmt.Printf("Logrus: %5d ns per request \n", x/10000)
	fmt.Printf("Zap:    %5d ns per request \n", y/10000)
	fmt.Printf("StdLog: %5d ns per request \n", z/10000)
	fmt.Printf("Glog:   %5d ns per request \n", w/10000)
}


/*
=====================
Logrus: 19305 ns per request
Zap:     1095 ns per request
StdLog:  7137 ns per request
Glog:   12070 ns per request
*/
```



### 2. zap 使用

+ sugar模式 (牺牲性能为代价,增强可用性)

```go
func main() {
	logger, _ := zap.NewProduction()
	defer logger.Sync()

	url := "https://www.liuvv.com"
	sugar := logger.Sugar()
	sugar.Infow("failed to fetch URL",
		"url", url,
		"attempt", 3,
		"backoff", time.Second,
	)

	sugar.Infof("Failed to fetch URL: %s", url)
}


// 注意 infow 和 infof 的调用区别

/*
{"level":"info","ts":1566623998.1506088,"caller":"log/main.go:15","msg":"failed to fetch URL","url":"https://www.liuvv.com","attempt":3,"backoff":1}
 
{"level":"info","ts":1566623998.15073,"caller":"log/main.go:20","msg":"Failed to fetch URL: https://www.liuvv.com"}
*/
```

+ logger 模式

```go
func main() {
	logger, _ := zap.NewProduction()
	defer logger.Sync()

	url := "https://www.liuvv.com"
	logger.Info("failed to fetch URL",
		zap.String("url", url),
		zap.Int("attempt", 3),
		zap.Duration("backoff", time.Second),
	)

	//logger.Infow() //没有此函数
	//logger.Infof() //没有此函数
}


/*
{"level":"info","ts":1566624270.4984472,"caller":"log/main.go:14","msg":"failed to fetch URL","url":"https://www.liuvv.com","attempt":3,"backoff":1}
*/
```



##### 2.1 zap序列化输出

```
zap.NewDevelopment() //格式化输出
zap.NewProduction() //json序列化输出
```



##### 2.2 输出到文件里

```go
func NewLogger() (*zap.Logger, error) {
  cfg := zap.NewProductionConfig()
  cfg.OutputPaths = []string{
    "/var/log/myproject/myproject.log",
  }
  return cfg.Build()
}
```



##### 2.3 输入到滚动文件里

Lumberjack用于将日志写入滚动文件。zap 不支持文件归档，如果要支持文件按大小或者时间归档，需要使用lumberjack，lumberjack也是zap官方推荐的。https://github.com/natefinch/lumberjack

```go
func main() {
	// lumberjack.Logger is already safe for concurrent use, so we don't need to
	// lock it.
	hook := &lumberjack.Logger{
		Filename:   "/tmp/foo.log", // 日志文件路径
		MaxSize:    500,            // 每个日志文件保存的最大尺寸 单位：M
		MaxBackups: 3,              // 日志文件最多保存多少个备份
		MaxAge:     28,             // 文件最多保存多少天
		Compress:   true,           // 是否压缩
	}
	core := zapcore.NewCore(
		zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig()),                       // 编码器配置
		zapcore.NewMultiWriteSyncer(zapcore.AddSync(os.Stdout), zapcore.AddSync(hook)), // 打印到控制台和文件
		zap.InfoLevel, // 日志级别
	)

	logger := zap.New(core)
	logger.Info("failed to fetch URL",
		zap.String("url", "https://www.liuvv.com"),
		zap.Int("attempt", 3),
		zap.Duration("backoff", time.Second),
	)
}
```



### 3. 参考资料

+ https://zhuanlan.zhihu.com/p/41991119