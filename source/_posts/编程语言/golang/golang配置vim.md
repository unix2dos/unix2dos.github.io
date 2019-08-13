---
title: golang配置vim
tags:
  - vim
  - golang
abbrlink: 3feff448
categories:
  - 编程语言
  - golang
date: 2016-11-16 22:14:49
---
### 配置文件和快速设置
- [.vimrc][1]
- [.zhsrc][2]
[1]: https://github.com/unix2dos/go-tutorial/blob/master/.vimrc
[2]: https://github.com/unix2dos/go-tutorial/blob/master/.zshrc

1. PlugClean
2. PlugInstall
3. 去到YCM里执行
```
./install.py --clang-completer --gocode-completer
```


### 安装插件管理器
```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### 安装插件
```bash
call plug#begin()
Plug 'fatih/vim-go' "go
Plug 'tomasr/molokai' "主题
Plug 'SirVer/ultisnips' "tab补全
Plug 'ctrlpvim/ctrlp.vim' "快速查文件
Plug 'Shougo/neocomplete.vim' "实时提示
Plug 'majutsushi/tagbar' "tagbar
Plug 'scrooloose/nerdtree' "导航
Plug 'vim-airline/vim-airline' "下面
call plug#end()
```
<!-- more -->

###  安装go tools需要的东西﻿
直接使用`:GoInstallBinaries`安装
如果网络不行(你懂的), 把代理把包都下载下来

```bash
git clone https://go.googlesource.com/tools  
```


### goTags需要安装ctags

```bash
brew install ctags
```
﻿

### 代码实时提示neocomplete, 需要vim支持lua
```bash
brew uninstall vim
brew install luajit
brew install vim --with-luajit
```


### Gocode autocomplete non imported packages
```
gocode set unimported-packages true
```