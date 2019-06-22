---
title: "tmux和oh_my_tmux的使用"
date: 2019-06-22 08:22:46
tags:
- tmux
- linux
---

tmux是一款优秀的终端复用软件，它比Screen更加强大. tmux之所以受人们喜爱，主要得益于以下功能：

- 丝滑分屏（split），虽然iTem2也提供了横向和竖向分屏功能，但这种分屏功能非常拙劣，完全等同于屏幕新开一个窗口，新开的pane不会自动进入到当前目录，也没有记住当前登录状态。这意味着如果我ssh进入到远程服务器时，iTem2新开的pane中，我依然要重新走一遍ssh登录的老路（omg）。tmux就不会这样，tmux窗口中，新开的pane，默认进入到之前的路径，如果是ssh连接，登录状态也依旧保持着，如此一来，我就可以随意的增删pane，这种灵活性，好处不言而喻。
- 保护现场（attach），即使命令行的工作只进行到一半，关闭终端后还可以重新进入到操作现场，继续工作。对于ssh远程连接而言，即使网络不稳定也没有关系，掉线后重新连接，可以直奔现场，之前运行中的任务，依旧在跑，就好像从来没有离开过一样；特别是在远程服务器上运行耗时的任务，tmux可以帮你一直保持住会话。如此一来，你就可以随时随地放心地进行移动办公，只要你附近的计算机装有tmux（没有你也可以花几分钟装一个），你就能继续刚才的工作。

<!-- more -->



### 1. tmux 安装

```bash
brew install tmux #mac

apt-get install tmux #linux
```



安装成功后, 需要再安装一个插件:   https://github.com/gpakosz/.tmux

```bash
cd
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```



以后配置修改   ~/.tmux.conf.local 即可. 



### 2. tmux 配置

```bash
tmux source-file ~/.tmux.conf # 刷新配置

set-option -g prefix2 `  # 设置一个不常用的`键作为指令前缀，按键更快些, 建议用 ctrl+a

set -g mouse on  # 最好关掉, 要不然影响iterm2自带鼠标选中
```



 Oh My Tmux! 

+  自动把 ctrl + a 当做第二个前缀
+  <prefix> m 切换鼠标开启状态
+ <prefix> e 自动打开配置, <prefix> r 刷新配置
+ <prefix> C-c  新建一个 Session
+ <prefix> C-h 和 <prefix> C-l 向左向右切换 window
+ <prefix> Tab 回到上一个 window
+ <prefix> - 和 <prefix> _  水平和垂直分屏
+ <prefix> h, <prefix> j, <prefix> k and <prefix> l    移动 panel
+ <prefix> + 让当前 panel 成为 window, 注意 再一次还能回到 panel
+  <prefix> H, <prefix> J, <prefix> K, <prefix> L     微调panel 大小(用的少)
+  <prefix> <  and <prefix> > 交换panel(用的少)



修改Oh My Tmux! 配置

+ tmux_conf_new_window_retain_current_path=true  window保持路径
+ tmux_conf_new_pane_reconnect_ssh=true  重新连接 ssh
+ tmux_conf_new_session_prompt=true  新建 session 输入名字
+ tmux_conf_theme_status_left=' ❐ #S '  左边状态栏精简



### 3. tmux 使用

tmux使用C/S模型构建，主要包括以下单元模块：

- server服务器。输入tmux命令时就开启了一个服务器。
- session会话。一个服务器可以包含多个会话
- window窗口。一个会话可以包含多个窗口。
- pane面板。一个窗口可以包含多个面板。



我习惯一个项目用一个 session, 一个工作区用一个 window, 快捷操作开始一个 panel. 如果刚开始记不住 tmux的操作, 一定多练习, 一定多用, 你会发现离不开它了.



#### 3.0 tmux 命令

```bash
tmux ls # 查看当前所有的session

tmux	# 新建一个无名称的会话, 可以用$再改名

tmux new -s demo # 新建一个名称为demo的会话, 

tmux attach -t session_name # 连接之前退出的session

tmux attach-session  # 快速进入 session

tmux kill-server  #关闭服务器，所有的会话都将关闭
```



#### 3.1 session操作

+ 新建 <prefix> C-c   或     :new 

+ 删除  :kill-session  或  tmux ls 以后 tmux kill-session -t 名字

+ 选择 s

+ 重命名 $

+ 退出  d



#### 3.2 window 操作



+ 新建   c
+ 关闭  ctrl+d 或   &
+ 列表  w   可切到其他 session

+ 重命名  ,

+ 跳跃  0-9
+ 向左 C-h  或   n
+ 向右 C-l  或   p



#### 3.3 panel 操作

+ 新建上下   - 或  "

+ 新建左右   _  或 %

+ 关闭  ctrl+d  或 x

+ 切换： 空格键

+ 移动    hjkl 键 或 上下左右键

+ 最大化  z

+ 变窗口  !   如果只是临时变 window, 用+

  

### 4. tmux TODO

1. off, 鼠标无法滚动

2.   on, 鼠标智能选中

3. 无论off, on   不是open   https://stackoverflow.com/questions/24761558/iterm2-cmd-click-to-open-file-doesnt-work-under-tmux-session