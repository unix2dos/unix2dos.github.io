---
title: golang命令行库cobra的使用
tags:
  - golang
  - cobra
categories:
  - 1-编程语言
  - golang
abbrlink: d50935d7
date: 2019-08-31 21:11:46
---

### 0. 前言

Cobra(眼镜蛇)是一个库，其提供简单的接口来创建强大现代的CLI接口，类似于git或者go工具。同时，它也是一个应用，用来生成个人应用框架，从而开发以Cobra为基础的应用。Docker源码中使用了Cobra。

Cobra基于三个基本概念`commands`,`arguments`和`flags`。其中commands代表行为，arguments代表数值，flags代表对行为的改变。

基本模型如下：

```bash
APPNAME COMMAND ARG --FLAG

# hugo是cmmands server是commands，port是flag
hugo server --port=1313

# clone是commands，URL是arguments，brae是flags
git clone URL --bare
```


<!-- more -->



### 1. 安装和使用

安装:

```bash
go get -u github.com/spf13/cobra/cobra
```



你的项目结构可能如下:

```bash
  ▾ appName/
    ▾ cmd/
        root.go
        version.go
        commands.go
      main.go
```



##### 1.1 main.go

In a Cobra app, typically the main.go file is very bare(裸露). It serves one purpose: initializing Cobra.

```go
package main

import (
  "appName/cmd"
)

func main() {
  cmd.Execute()
}
```



##### 1.2 rootcmd

```go
var rootCmd = &cobra.Command{
  Use:   "hugo",
  Short: "Hugo is a very fast static site generator",
  Long: `A Fast and Flexible Static Site Generator built with
                love by spf13 and friends in Go.
                Complete documentation is available at http://hugo.spf13.com`,
  Run: func(cmd *cobra.Command, args []string) {
    // Do Stuff Here
  },
}

func Execute() {
  if err := rootCmd.Execute(); err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
}
```



##### 1.3 additional commands

```go
package cmd

import (
  "fmt"

  "github.com/spf13/cobra"
)

func init() {
  rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
  Use:   "version",
  Short: "Print the version number of Hugo",
  Long:  `All software has versions. This is Hugo's`,
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
  },
}
```



可以简单执行下面命令查看效果

```bash
go run main.go help
go run main.go version
```



### 3. cobra 生成器

在文件夹github.com/spf13/cobra/cobra下使用go install, 生成 cobra命令

命令`cobra init [yourApp]`将会创建初始化应用，yourApp 是你的项目名称。它会在你的 GOPATH 目录下面生成项目。最新的操作方式:

```bash
cd /Users/liuwei/golang/src/github.com/unix2dos/golangTest
cobra init --pkg-name=github.com/unix2dos/golangTest yourApp
```



接下来我们用`cobra add`来添加一些子命令。在你项目的目录下，运行下面这些命令：

```bash
cobra add serve
cobra add config
cobra add create -p 'configCmd'
```

这样以后，你就可以运行上面那些 app serve 之类的命令了。项目目录如下：

```bash
▾ app/
  ▾ cmd/
      serve.go
      config.go
      create.go
    main.go
```



### 4. 使用Flags

cobra 有两种 flag，一个是全局变量，一个是局部变量。全局什么意思呢，就是所以子命令都可以用。局部的只有自己能用。先看全局的：

```go
RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra_exp1.yaml)")
```

在看局部的：

```go
RootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
```

区别就在 RootCmd 后面的是 Flags 还是 PersistentFlags。



### 5. 参考资料

+ [Golang之使用Cobra](https://o-my-chenjian.com/2017/09/20/Using-Cobra-With-Golang/)

+ https://www.kancloud.cn/liupengjie/go/1010466