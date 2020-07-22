---
title: mac的homebrew使用
tags:
  - mac
categories:
  - 4-mac系统
  - 软件
abbrlink: be2ea1c8
date: 2019-04-07 00:00:00
---

Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。

<!-- more -->

Homebrew 会将软件包安装到独立目录，并将其文件软链接至 /usr/local 。

```bash
cd /usr/local
find Cellar
Cellar/wget/1.16.1
Cellar/wget/1.16.1/bin/wget
Cellar/wget/1.16.1/share/man/man1/wget.1

$ ls -l bin
bin/wget -> ../Cellar/wget/1.16.1/bin/wget
```



# 1. 安装使用

去官网 https://brew.sh/ 看最新下载信息:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

常使用命令

```bash
brew --version 或者 brew -v # 显示brew版本信息

brew list   #显示所有的已安装的软件
brew update #自动升级homebrew
brew upgrade  #升级所有已过时的软件，即列出的以过时软件
brew upgrade <formula> #升级指定的软件
```



# 2. 查看源更改源

对于 homebrew，需要替换的是4个模块的镜像

+ Homebrew（brew --repo）

+ Homebrew Core（brew --repo homebrew/core）

+ Homebrew Cask（brew --repo homebrew/cask）

+ Homebrew-bottles

替换: 

```bash
#替换 Homebrew
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

#替换 Homebrew Core
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

#替换 Homebrew Cask
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

#替换 Homebrew-bottles
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc

# 更新
brew update
```



# 3. 参考资料

+ https://brew.sh/
+ https://www.zhihu.com/question/31360766/answer/749386652