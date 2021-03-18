---
title: fzf模糊搜索神器的安装和使用
tags:
  - linux
  - fzf
categories:
  - 2-linux系统
  - shell
abbrlink: a0700771
date: 2021-03-18 00:00:00
---

fzf是一个通用的命令行模糊查找器, 通过输入模糊的关键词就可以定位文件或文件夹。结合其他工具(比如rg)可以完成非常多的工作，在工作中可以大幅提高你的工作效率。

fzf可以用于文件、命令历史记录、进程、主机名、书签、git提交等。

<!-- more -->

# 1. fzf使用

### 1.1 安装


```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
source ~/.zshrc  
```

### 1.2 使用

安装后, 可以执行下`fzf`, 先体验下, 另外 fzf 重写了 `ctrl+r` 搜索历史命令

![image-20210318231127907](fzf%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2%E7%A5%9E%E5%99%A8%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/image-20210318231127907.png)

```bash
vim $(fzf)  # 搜索后, 回车直接用 vi 打开
vim $(fzf --height 40%) # 高度40%打开
```

+ 搜索过程中, CTRL-J 和 CTRL-K 向上翻和向下翻

+ bash和zsh的模糊完备, 默认触发是`**`,  例如: `vim **<TAB>`, 或 `cd **<TAB>`, 或 `ssh **<TAB>`, 简直好用到飞起.

  ![image-20210318000439297](fzf%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2%E7%A5%9E%E5%99%A8%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/1.png)

+ 一边查一边预览

  ```bash
  fzf --preview 'cat {}'
  ```

+ 可以配合管道使用

  ```bash
  ps -ef | fzf
  seq 100 | fzf
  history | fzf
  ```

  

### 1.3 搜索语法

| Token     | Match type                 | Description                          |
| --------- | -------------------------- | ------------------------------------ |
| `sbtrkt`  | fuzzy-match                | Items that match `sbtrkt`            |
| `'wild`   | exact-match (quoted)       | Items that include `wild`            |
| `^music`  | prefix-exact-match         | Items that start with `music`        |
| `.mp3$`   | suffix-exact-match         | Items that end with `.mp3`           |
| `!fire`   | inverse-exact-match        | Items that do not include `fire`     |
| `!^music` | inverse-prefix-exact-match | Items that do not start with `music` |
| `!.mp3$`  | inverse-suffix-exact-match | Items that do not end with `.mp3`    |



### 1.4 和tmux结合

fzf 安装后自带一个 fzf-tmux, 但是会新开一个 panel ,并不是很好用,  建议以弹窗形成弹出

参考: https://www.liuvv.com/p/1104a363.html#3-fzf-tmux



### 1.5 打开文件

zsh 增加以下函数, ctrl-o 用 open 打开, ctrl-e 用 vim 打开

```bash
# Modified version where you can press
#   - CTRL-O to open with `open` command,
#   - CTRL-E or Enter key to open with the $EDITOR
fo() {
  IFS=$'\n' out=("$(fzfp --preview 'cat {}' --query="$1" --exit-0 --expect=ctrl-o,ctrl-e)")
  key=$(head -1 <<< "$out")
  file=$(head -2 <<< "$out" | tail -1)
  if [ -n "$file" ]; then
    [ "$key" = ctrl-o ] && open "$file" || ${EDITOR:-vim} "$file"
  fi
}
```

![image-20210318231156159](fzf%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2%E7%A5%9E%E5%99%A8%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/image-20210318231156159.png)

### 1.6 切换目录

zsh 增加以下函数

```bash
# cd to selected directory
fd() {
  local dir
  dir=$(find ${1:-.} -path '*/\.*' -prune \
                  -o -type d -print 2> /dev/null | fzfp +m) &&
  cd "$dir"
}
```



### 1.7 搜索文件内容

zsh 增加以下函数, 需要配合 `rg`命令

```bash
#find-in-file - usage: fif <searchTerm>
fif() {
  if [ ! "$#" -gt 0 ]; then echo "Need a string to search for!"; return 1; fi
  rg --files-with-matches --no-messages "$1" | fzf --preview "highlight -O ansi -l {} 2> /dev/null | rg --colors 'match:bg:yellow' --ignore-case --pretty --context 10 '$1' || rg --ignore-case --pretty --context 10 '$1' {}"
}
```



# 2. vim使用fzf

### 2.1 安装

```bash
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } } "极限搜索文件
Plug 'junegunn/fzf.vim'
nnoremap <leader>fo :Files<CR>"映射
nnoremap <leader>fif :Rg<CR> "映射
```

### 2.2 使用

`:Files`

![image-20210318230744605](fzf%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2%E7%A5%9E%E5%99%A8%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/2.png)

`:Rg`

![image-20210318230855552](fzf%E6%A8%A1%E7%B3%8A%E6%90%9C%E7%B4%A2%E7%A5%9E%E5%99%A8%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/3.png)



# 3. 参考资料

+ https://github.com/junegunn/fzf
+ https://github.com/junegunn/fzf/wiki/Examples
+ https://github.com/junegunn/fzf.vim