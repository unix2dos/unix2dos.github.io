---
title: vim实用插件的使用
tags:
  - vim
categories:
  - 2-linux系统
  - vim
abbrlink: 999f6ee6
date: 2021-03-20 00:00:00
---

工欲善其事必先利其器,  记录一些 vim 常用的插件。

<!-- more -->

# 1. 插件管理vim-plug

项目地址: https://github.com/junegunn/vim-plug

### 1.1 安装

``` bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### 1.2 使用

```yml
call plug#begin('~/.vim/plugged')
Plug 'tpope/vim-sensible'
Plug 'junegunn/seoul256.vim'
call plug#end()
```

+ 安装插件: PlugInstall
+ 卸载插件: PlugClean



# 2. 主题gruvbox

项目地址: https://github.com/morhetz/gruvbox

### 2.1 安装

```yml
Plug 'morhetz/gruvbox'
```



# 3. 状态栏airline

项目地址: https://github.com/vim-airline/vim-airline

### 3.1 安装

```yml
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
let g:airline#extensions#tabline#enabled = 1
let g:airline_theme='simple' " https://github.com/vim-airline/vim-airline/wiki/Screenshots
```



# 4. 模糊搜索fzf 

### 4.1 安装

```yml
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } } "极限搜索文件
Plug 'junegunn/fzf.vim'
```



# 5. 极限跳转easymotion

### 5.1 安装

```yml
Plug 'easymotion/vim-easymotion' "极速搜索跳转
let g:EasyMotion_startofline = 0 " keep cursor column when JK motion
map  / <Plug>(easymotion-sn)
omap / <Plug>(easymotion-tn)
map  n <Plug>(easymotion-next)
map  N <Plug>(easymotion-prev)
map <Leader>l <Plug>(easymotion-lineforward)
map <Leader>j <Plug>(easymotion-j)
map <Leader>k <Plug>(easymotion-k)
map <Leader>h <Plug>(easymotion-linebackward)
```



# 10. 参考资料

+ https://github.com/topics/vim

