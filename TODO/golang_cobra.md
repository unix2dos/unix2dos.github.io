```
github.com/spf13/cobra
```



命令行库cobra
简介
cobra 回一个用来创建强大的现代CLI命令行的golang库, 也会一个生成程序应用和命令行文件的程序

功能
1 简单的子命令行模式, 如app server, app fetch

2 完全兼容posix命令行模式

3 嵌套子命令subcommand

4 支持全局, 局部, 串联flags

5 使用cobra很容易生成应用程序和命令, 会用cobra create appname 和cobra add cmdname

6 如果命令输入错误, 智能提示

7 自动识别-h --help帮助flag

8 自动横撑应用程序在bah下命令自动完成功能

9 自动生成应用程序的man手册

10 命令行别名

11自定义help和uage信息

12 可选的紧密集成的viper apps

安装
go get -v github.com/spf13/cobra/cobra

建立项目
cobra.exe只能在GOPATH目录下执行
src> ..\bin\cobra.exe init demo

在src目录下会生成一个demo的文件夹，如下：

▾ demo
    ▾ cmd/
        root.go
    main.go
如何实现没有子命令的CLIs程序
接下来就是可以继续demo的功能设计了。例如我在demo下面新建一个包，名称为imp。如下：

▾ demo
    ▾ cmd/
        root.go
    ▾ imp/
        imp.go
        imp_test.go
    main.go
imp.go文件的代码如下：

package imp

import(
    "fmt"
)

func Show(name string, age int) {
    fmt.Printf("My Name is %s, My age is %d\n", name, age)
}
从源代码来看cmd包进行了一些初始化操作并提供了Execute接口。十分简单，其中viper是cobra集成的配置文件读取的库，这里不需要使用，我们可以注释掉（不注释可能生成的应用程序很大约10M，这里没哟用到最好是注释掉）。cobra的所有命令都是通过cobra.Command这个结构体实现的。为了实现demo功能，显然我们需要修改RootCmd。修改后的代码如下：

package cmd

import (
	"fmt"
	"os"
	homedir "github.com/mitchellh/go-homedir"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
	"demo/imp"
)

var cfgFile string

var name string
var age int

var RootCmd = &cobra.Command{
	Use:   "demo",
	Short: "A test demo",
	Long:  `Demo is a test appcation for print things`,
	Run: func(cmd *cobra.Command, args []string) {
		if len(name) == 0 {
			cmd.Help()
			return
		}
		imp.Show(name, age)
	},
}

func init() { 
	cobra.OnInitialize(initConfig)
	RootCmd.Flags().StringVarP(&name, "name", "n", "", "person's name")
	RootCmd.Flags().IntVarP(&age, "age", "a", 0, "person's age")

}
如何实现带有子命令的CLIs程序
在执行cobra.exe init demo之后，继续使用cobra为demo添加子命令test：

src\demo>..\..\bin\cobra add test
test created at C:\Users\liubo5\Desktop\transcoding_tool\src\demo\cmd\test.go
在src目录下demo的文件夹下生成了一个cmd\test.go文件，如下：

▾ demo
    ▾ cmd/
        root.go
        test.go
    main.go