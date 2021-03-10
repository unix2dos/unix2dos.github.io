---
title: "tmux的浮动弹窗popup功能"
date: 2021-03-10 00:00:01
tags:
- tmux
---

# 1. 介绍

tmux 已经支持 popup功能, 但是暂时还没有发布到 stable release 版本, 所以需要使用的需要在开发分支上编译使用, 即tmux 版本需要>=3.2

![image-20210310231447891](tmux%E7%9A%84%E6%B5%AE%E5%8A%A8%E5%BC%B9%E7%AA%97popup%E5%8A%9F%E8%83%BD/image-20210310231447891.png)

<!-- more -->

# 2. 使用

### 2.1 升级 tmux

确认下 tmux 版本, 如果小于3.2就开始升级.

```bash
tmux -V
```

https://github.com/tmux/tmux/releases 下载最新的 release

```bash
./configure && make
sudo make install
```

注意: 升级后如果开启了, 保持 session会卡死, 需要 `tmux kill-server `后, 再来一次 (巨坑!!!)

创建两个别名:


```bash
alias p='tmux popup -w 80% -h 80%' 
alias pp='tmux popup -w 90% -h 90%  "tmux attach -t popup || tmux new -s popup"'
```



# 3. fzf-tmux

https://github.com/kevinhwang91/fzf-tmux-script/





#  4. tmux-fzf 

Use fzf to manage your tmux work environment!

https://github.com/sainnhe/tmux-fzf


set -g @plugin 'sainnhe/tmux-fzf'

Reload configuration, then press prefix + I.

To launch tmux-fzf, press prefix + F (Shift+F).



# 5. 参考资料:

+ https://github.com/tmux/tmux/issues/1842
+ https://blog.meain.io/2020/tmux-flating-scratch-terminal/
+ https://github.com/kevinhwang91/fzf-tmux-script
+ https://github.com/sainnhe/tmux-fzf

