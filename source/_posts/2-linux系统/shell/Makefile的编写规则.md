---
title: makefile的编写规则
tags:
  - makefile
categories:
  - 2-linux系统
  - shell
abbrlink: 4cf47ff4
date: 2019-11-21 21:11:46
---



### 1. Makefile 介绍

Makefile文件由一系列规则（rules）构成。每条规则的形式如下。

```bash
<target> : <prerequisites> 
[tab]  <commands>
```

上面第一行冒号前面的部分，叫做"目标"（target），冒号后面的部分叫做"前置条件"（prerequisites）；第二行必须由一个tab键起首，后面跟着"命令"（commands）。

"目标"是必需的，不可省略；"前置条件"和"命令"都是可选的，但是两者之中必须至少存在一个。

每条规则就明确两件事：构建目标的前置条件是什么，以及如何构建。

<!-- more -->

##### 1.1 目标（target）

一个目标（target）就构成一条规则。目标通常是文件名，指明Make命令所要构建的对象，目标可以是一个文件名，也可以是多个文件名，之间用空格分隔。

除了文件名，目标还可以是某个操作的名字，这称为"伪目标"（phony target）。

```makefile
clean:
      rm *.o
```

上面代码的目标是clean，它不是文件名，而是一个操作的名字，属于"伪目标 "，作用是删除对象文件。

```bash
$ make  clean
```

但是，如果当前目录中，正好有一个文件叫做clean，那么这个命令不会执行。因为Make发现clean文件已经存在，就认为没有必要重新构建了，就不会执行指定的rm命令。



为了避免这种情况，可以明确声明clean是"伪目标"，写法如下。

```makefile
.PHONY: clean
clean:
        rm *.o temp
```



声明clean是"伪目标"之后，make就不会去检查是否存在一个叫做clean的文件，而是每次运行都执行对应的命令。



如果Make命令运行时没有指定目标，默认会执行Makefile文件的第一个目标。

```bash
make
```



##### 1.2 前置条件（prerequisites）

前置条件通常是一组文件名，之间用空格分隔。它指定了"目标"是否重新构建的判断标准：只要有一个前置文件不存在，或者有过更新（前置文件的last-modification时间戳比目标的时间戳新），"目标"就需要重新构建。

```makefile
result.txt: source.txt
    cp source.txt result.txt
```

上面代码中，构建 result.txt 的前置条件是 source.txt 。如果当前目录中，source.txt 已经存在，那么`make result.txt`可以正常运行，否则必须再写一条规则，来生成 source.txt 。

```makefile
source.txt:
    echo "this is the source" > source.txt
```



上面代码中，source.txt后面没有前置条件，就意味着它跟其他文件都无关，只要这个文件还不存在，每次调用`make source.txt`，它都会生成。

```bash
$ make result.txt
$ make result.txt
```

上面命令连续执行两次`make result.txt`。第一次执行会先新建 source.txt，然后再新建 result.txt。第二次执行，Make发现 source.txt 没有变动（时间戳晚于 result.txt），就不会执行任何操作，result.txt 也不会重新生成。

##### 1.3 命令（commands）

命令（commands）表示如何更新目标文件，由一行或多行的Shell命令组成。它是构建"目标"的具体指令，它的运行结果通常就是生成目标文件。

每行命令之前必须有一个tab键。需要注意的是，每行命令在一个单独的shell中执行。这些Shell之间没有继承关系。

```makefile
var-lost:
    export foo=bar
    echo "foo=[$$foo]"
```

上面代码执行后（`make var-lost`），取不到foo的值。因为两行命令在两个不同的进程执行。一个解决办法是将两行命令写在一行，中间用分号分隔。

```makefile
var-kept:
    export foo=bar; echo "foo=[$$foo]"
```

另一个解决办法是在换行符前加反斜杠转义。

```makefile
var-kept:
    export foo=bar; \
    echo "foo=[$$foo]"
```





### 2. Makefile文件的语法



##### 2.1 井号（#）

在Makefile中表示注释。



##### 2.2 回声（echoing）

正常情况下，make会打印每条命令，然后再执行，这就叫做回声（echoing）。

在命令的前面加上@，就可以关闭回声。

```makefile
test:
    # 这是测试

test:
    @# 这是测试
```



##### 2.3 通配符

通配符（wildcard）用来指定一组符合条件的文件名。Makefile 的通配符与 Bash 一致，主要有星号（*）、问号（？）和 [...] 。比如， *.o 表示所有后缀名为o的文件。

```makefile
clean:
        rm -f *.o
```



##### 2.4 模式匹配

Make命令允许对文件名，进行类似正则运算的匹配，主要用到的匹配符是%。比如，假定当前目录下有 f1.c 和 f2.c 两个源码文件，需要将它们编译为对应的对象文件。

```
%.o: %.c

等同于下面的写法。

f1.o: f1.c
f2.o: f2.c
```

使用匹配符%，可以将大量同类型的文件，只用一条规则就完成构建。



##### 2.5 变量和赋值符

Makefile 允许使用等号自定义变量。

```makefile
txt = Hello World
test:
    @echo $(txt)
```

上面代码中，变量 txt 等于 Hello World。调用时，变量需要放在 $( ) 之中。



调用Shell变量，需要在美元符号前，再加一个美元符号，这是因为Make命令会对美元符号转义。

```makefile
test:
    @echo $$HOME
```



有时，变量的值可能指向另一个变量。

```makefile
v1 = $(v2)
```

上面代码中，变量 v1 的值是另一个变量 v2。这时会产生一个问题，v1 的值到底在定义时扩展（静态扩展），还是在运行时扩展（动态扩展）？如果 v2 的值是动态的，这两种扩展方式的结果可能会差异很大。

为了解决类似问题，Makefile一共提供了四个赋值运算符 （=、:=、？=、+=），它们的区别请看[StackOverflow](http://stackoverflow.com/questions/448910/makefile-variable-assignment)。

```bash
VARIABLE = value
# 在执行时扩展，允许递归扩展。

VARIABLE := value
# 在定义时扩展。

VARIABLE ?= value
# 只有在该变量为空时才设置值。

VARIABLE += value
# 将值追加到变量的尾端。
```



##### 2.6 内置变量（Implicit Variables）

Make命令提供一系列内置变量，比如，`$(CC)` 指向当前使用的编译器，`$(MAKE)` 指向当前使用的Make工具。这主要是为了跨平台的兼容性，详细的内置变量清单见[手册](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)。

```makefile
output:
    $(CC) -o output input.c
```



##### 2.7 自动变量（Automatic Variables）

Make命令还提供一些自动变量，它们的值与当前规则有关。主要有以下几个。

+ ``$@``

  `$@`指代当前目标，就是Make命令当前构建的那个目标。比如，`make foo`的`$@` 就指代foo。

  ```bash
  a.txt b.txt: 
      touch $@
      
  #等同于下面的写法。    
   
  a.txt:
      touch a.txt
  b.txt:
      touch b.txt
  ```

+ `$<`

  `$<` 指代第一个前置条件。比如，规则为 t: p1 p2，那么`$<` 就指代p1。

  ```bash
  a.txt: b.txt c.txt
      cp $< $@ 
      
  # 等同于下面的写法。
  
  a.txt: b.txt c.txt
      cp b.txt a.txt 
  ```

  

+ `$?`

  `$?` 指代比目标更新的所有前置条件，之间以空格分隔。比如，规则为 t: p1 p2，其中 p2 的时间戳比 t 新，`$?`就指代p2。

  

+ `$^`

  `$^` 指代所有前置条件，之间以空格分隔。比如，规则为 t: p1 p2，那么 `$^` 就指代 p1 p2 。

  

+ `$*`

  `$*` 指代匹配符 % 匹配的部分， 比如% 匹配 f1.txt 中的f1 ，`$*` 就表示 f1。

  

+ `$(@D)` 和 `$(@F)`

  `$(@D)` 和 `$(@F)` 分别指向 `$@` 的目录名和文件名。比如，`$@`是 src/input.c，那么`$(@D)` 的值为 src ，`$(@F)` 的值为 input.c。

  

+ `$(<D)` 和 `$(<F)`

  `$(<D)` 和 `$(<F)` 分别指向 `$<` 的目录名和文件名。



下面是自动变量的一个例子。

```makefile
dest/%.txt: src/%.txt
    @[ -d dest ] || mkdir dest
    cp $< $@
```

上面代码将 src 目录下的 txt 文件，拷贝到 dest 目录下。首先判断 dest 目录是否存在，如果不存在就新建，然后，`$<` 指代前置文件（src/%.txt）， `$@` 指代目标文件（dest/%.txt）。



##### 2.8 判断和循环

Makefile使用 Bash 语法，完成判断和循环。

```makefile
ifeq ($(CC),gcc)
  libs=$(libs_for_gcc)
else
  libs=$(normal_libs)
endif
```

上面代码判断当前编译器是否 gcc ，然后指定不同的库文件。



```makefile
LIST = one two three
all:
    for i in $(LIST); do \
        echo $$i; \
    done

# 等同于

all:
    for i in one two three; do \
        echo $i; \
    done
```

上面代码的运行结果。

```bash
one
two
three
```



##### 2.9 函数

Makefile 还可以使用函数，格式如下。

```bash
$(function arguments)
# 或者
${function arguments}
```



Makefile提供了许多[内置函数](http://www.gnu.org/software/make/manual/html_node/Functions.html)，可供调用。下面是几个常用的内置函数。

+ shell 函数

  shell 函数用来执行 shell 命令

  ```makefile
  srcfiles := $(shell echo src/{00..99}.txt)
  ```

  

+ wildcard 函数

  wildcard 函数用来在 Makefile 中，替换 Bash 的通配符。

  ```makefile
  srcfiles := $(wildcard src/*.txt)
  ```

  

+ subst 函数

  subst 函数用来文本替换，格式如下。

  ```makefile
  $(subst from,to,text)
  ```

  下面的例子将字符串"feet on the street"替换成"fEEt on the strEEt"。

  ```makefile
  $(subst ee,EE,feet on the street)
  ```

  

+ patsubst函数

  patsubst 函数用于模式匹配的替换，格式如下。

  ```makefile
  $(patsubst pattern,replacement,text)
  ```

  下面的例子将文件名"x.c.c bar.c"，替换成"x.c.o bar.o"。

  ```makefile
  $(patsubst %.c,%.o,x.c.c bar.c)
  ```

  

+ 替换后缀名

  替换后缀名函数的写法是：变量名 + 冒号 + 后缀名替换规则。它实际上patsubst函数的一种简写形式。

  ```makefile
  min: $(OUTPUT:.js=.min.js)
  ```

  上面代码的意思是，将变量OUTPUT中的后缀名 .js 全部替换成 .min.js 。



### 3. Makefile 的实例

##### 3.1  删除

```makefile
.PHONY: cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
        rm program

cleanobj :
        rm *.o

cleandiff :
        rm *.diff
```

上面代码可以调用不同目标，删除不同后缀名的文件，也可以调用一个目标（cleanall），删除所有指定类型的文件。

##### 3.2 编译C语言项目

```makefile
edit : main.o kbd.o command.o display.o 
    cc -o edit main.o kbd.o command.o display.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h
    cc -c display.c

clean :
     rm edit main.o kbd.o command.o display.o

.PHONY: edit clean
```



##### 3.3 项目

```makefile
.SILENT :
.PHONY : dep vet clean dist package test

NAME := cistern
PRE := oc
ROOF := fhyx.tech/oceans/$(NAME)

WITH_ENV = env `cat .env 2>/dev/null | xargs`
UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
    HOST := $(shell scutil --get LocalHostName)
else
    HOST := $(shell hostname)
endif

DATE := $(shell date '+%Y%m%dT%H%M')
STAMP := $(shell date +%s)
USER := $(shell echo ${USER})
TAG:=$(shell git describe --tags --always)
LDFLAGS:=-X $(ROOF)/settings.Name=$(NAME) -X $(ROOF)/cmd.version=$(TAG) -X $(ROOF)/cmd.built=$(DATE) -X $(ROOF)/cmd.buildStamp=$(STAMP) -X $(ROOF)/cmd.buildUser=$(USER) -X $(ROOF)/cmd.buildHost=$(HOST)

COMMANDS = vet clean dist
.PHONY: $(COMMANDS)

main:
	echo "Building $(NAME)"
	go build -ldflags "$(LDFLAGS)" .

help:
	@echo "commands: $(COMMANDS)"

all: clean $(COMMANDS)

vet:
	echo "Checking ."
	go vet -vettool=$(which shadow) -atomic -bool -copylocks -nilfunc -printf -rangeloops -unreachable -unsafeptr -unusedresult ./...

clean:
	echo "Cleaning dist"
	rm -rf dist
	rm -f $(NAME) $(NAME)-*

dist/linux_amd64/$(NAME): $(SOURCES)
	echo "Building $(NAME) of linux"
	mkdir -p dist/linux_amd64 && GOOS=linux GOARCH=amd64 go build -ldflags "$(LDFLAGS) -s -w" -o dist/linux_amd64/$(PRE)-$(NAME) $(ROOF)

dist/darwin_amd64/$(NAME): $(SOURCES)
	echo "Building $(NAME) of darwin"
	mkdir -p dist/darwin_amd64 && GOOS=darwin GOARCH=amd64 go build -ldflags "$(LDFLAGS) -w" -o dist/darwin_amd64/$(PRE)-$(NAME) $(ROOF)

dist: clean dist/linux_amd64/$(NAME) dist/darwin_amd64/$(NAME)

package: dist
	echo "Packaging $(NAME)"
	ls dist/linux_amd64 | xargs tar -cvJf $(NAME)-linux-amd64-$(TAG).tar.xz -C dist/linux_amd64

.PHONY: binary-deploy
binary-deploy:
	@echo "copy binary to earth"
	@scp dist/linux_amd64/??-* earth:dist/linux_amd64/

.PHONY: package-upload
package-upload:
	@echo "copy package.tar.?z to venus"
	@scp *-linux-amd64-*.tar.?z gopkg:/var/www/gopkg/

docker-build: dist/linux_amd64/$(NAME)
	echo "Building docker image"
	cp -rf Dockerfile* dist/
	docker build -t fhyx/cistern:$(TAG) dist/
	docker tag fhyx/cistern:$(TAG) fhyx/cistern:latest
.PHONY: $@
```



```go
package cmd

import (
	"fmt"
	"runtime"

	"github.com/spf13/cobra"
)

var (
	version   = "dev"
	built     = "N/A"
	buildUser = "None"
	name      = "cistern"
)

var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "Print the version number",
	Long:  ``,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Printf("%s %s built %s by %s (%s %s-%s)\n", name, version, built, buildUser, runtime.Version(), runtime.GOOS, runtime.GOARCH)
	},
}

func init() {
	RootCmd.AddCommand(versionCmd)
}

func inDevelop() bool {
	return version == "dev"
}
```



### 4. 参考资料

+ http://www.ruanyifeng.com/blog/2015/02/make.html
+ https://blog.csdn.net/ruglcc/article/details/7814546
+ http://www.gnu.org/software/make/manual/html_node/Special-Targets.html#Special-Targets
