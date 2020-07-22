---
title: node版本管理nvm的安装和使用
tags:
  - nodejs
abbrlink: df22a20c
categories:
  - 1-编程语言
  - nodejs
date: 2019-05-10 20:02:00
---



我们可能同时在进行2个项目，而2个不同的项目所使用的node版本又是不一样的，或者是要用更新的node版本进行试验和学习。这种情况下，对于维护多个版本的node将会是一件非常麻烦的事情，而nvm就是为解决这个问题而产生的，他可以方便的在同一台设备上进行多个node版本之间切换，而这个正是nvm的价值所在。



### 1. nodejs，npm，nvm之间的区别

+ nodejs：在项目开发时的所需要的代码库

+ npm：nodejs 包管理工具。在安装的 nodejs 的时候，npm 也会跟着一起安装，它是包管理工具。npm 管理 nodejs 中的第三方插件

+ nvm：nodejs 版本管理工具。也就是说：一个 nvm 可以管理很多 node 版本和 npm 版本。



<!-- more -->

### 2. nvm的安装:

如果在安装nvm之前就安装了node, 那么最好在安装之前清理下全局的node环境.

```shell
npm ls -g --depth=0 # 查看已经安装在全局的模块，以便删除这些全局模块后再按照不同的 node 版本重新进行全局安装

sudo rm -rf /usr/local/lib/node_modules # 删除全局 node_modules 目录

sudo rm -rf ~/.npm/ # 删除模块缓存目录

sudo rm /usr/local/bin/node # 删除 node

cd  /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs rm # 删除全局 node 模块注册的软链
```



安装: https://github.com/nvm-sh/nvm

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

重新打开一个终端输入 `nvm`即可



### 3. nvm常用指令

```shell
nvm version #查看当前的版本

nvm ls-remote #列出所有可安装的版本

nvm install <version> #安装指定的版本，如nvm install v8.14.0

nvm uninstall <version> #卸载指定的版本

nvm ls #列出所有已经安装的版本

nvm use <version> #切换使用指定的版本

nvm current #显示当前使用的版本

nvm alias default <version> #设置默认node版本

```



#### 3.1 安装最新稳定版 node

```shell
nvm install stable  
```



#### 3.2 node被安装在哪里了呢?

在终端我们可以使用which node来查看我们的node被安装到了哪里，这里终端打印出来的地址其实是你当前使用的node版本快捷方式的地址。

```
$ which node
/Users/liuwei/.nvm/versions/node/v12.2.0/bin/node


$ which npm
/Users/liuwei/.nvm/versions/node/v12.2.0/bin/npm
```



### 4. nvm 和 n  的区别

在 node 的版本管理工具中，nvm 自然声名远扬，然而我们也不能忘了来自 TJ 的 n。这两种，是目前最主流的方案。

关于这两个工具如何安装和使用，这里不再赘言，请见它们各自的主页：

- [creationix/nvm](https://github.com/creationix/nvm)
- [tj/n](https://github.com/tj/n)

接下来我们着重关注一下 nvm 和 n 的运作机制和特性。

#### 4.1 n

n 是一个需要全局安装的 npm package。

```shell
npm install -g n
```


这意味着，我们在使用 n 管理 node 版本前，首先需要一个 node 环境。我们或者用 Homebrew 来安装一个 node，或者从官网下载 pkg 来安装，总之我们得先自己装一个 node —— n 本身是没法给你装的。

然后我们可以使用 n 来安装不同版本的 node。

在安装的时候，n 会先将指定版本的 node 存储下来，然后将其复制到我们熟知的路径/usr/local/bin，非常简单明了。当然由于 n 会操作到非用户目录，所以需要加 sudo 来执行命令。

所以这样看来，n 在其实现上是一个非常易理解的方案。



#### 4.2 nvm

我们再来看 nvm。不同于 n，nvm 不是一个 npm package，而是一个独立软件包。这意味着我们需要单独使用它的安装逻辑：

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash(zsh)# 注意后面的bash环境
```



然后我们可以使用 nvm 来安装不同版本的 node。

在安装的时候，nvm 将不同的 node 版本存储到 ~/.nvm/<version>/ 下，然后修改$PATH，将指定版本的 node 路径加入，这样我们调用的 node 命令即是使用指定版本的 node。

nvm 显然比 n 要复杂一些，但是另一方面，由于它是一个独立软件包，因此它和 node 之间的关系看上去更合乎逻辑：nvm 不依赖 node 环境，是 node 依赖 nvm；而不像 n 那样产生类似循环依赖的问题。

