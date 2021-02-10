---
title: "alacritty替代iterm2"
date: 2021-02-10 00:00:00
tags:
- alacritty
- iterm2
---

# 1. alacritty

### 1.1 介绍

iterm2 无疑是所有平台里功能最强的终端，遗憾的是目前 GPU 加速并不完美。

alacritty是目前性能最强的终端之一. 它使用GPU进行渲染，可以做到其他启动器无法实现的性能优化。

尤其 `tmux`配合`alacritty`, 使用下来比 iTerm2 更快更顺手更省电。

<!-- more -->

### 1.2 安装

https://github.com/alacritty/alacritty/releases

下载对应安装包安装即可



# 2. 配置

### 2.1 配置方法

Alacritty doesn't create the config file for you, but it looks for one in the following locations:

```bash
$XDG_CONFIG_HOME/alacritty/alacritty.yml
$XDG_CONFIG_HOME/alacritty.yml
$HOME/.config/alacritty/alacritty.yml
$HOME/.alacritty.yml
```



下载 github 的模板

```bash
mv ~/Downloads/alacritty.yml ~/.alacritty.yml
```

修改保存即生效



### 2.2 我的配置

```yml
# 环境变量
env:
  TERM: alacritty

# 字体
font:
  normal:
    family: MesloLGS NF
  size: 17.0


# 吸血鬼配色
colors:
  primary:
    background: '0x282a36'
    foreground: '0xf8f8f2'
  cursor:
    text: CellBackground
    cursor: CellForeground
  vi_mode_cursor:
    text: CellBackground
    cursor: CellForeground
  search:
    matches:
      foreground: '0x44475a'
      background: '0x50fa7b'
    focused_match:
      foreground: '0x44475a'
      background: '0xffb86c'
    bar:
      background: '0x282a36'
      foreground: '0xf8f8f2'
  line_indicator:
    foreground: None
    background: None
  selection:
    text: CellForeground
    background: '0x44475a'
  normal:
    black:   '0x000000'
    red:     '0xff5555'
    green:   '0x50fa7b'
    yellow:  '0xf1fa8c'
    blue:    '0xbd93f9'
    magenta: '0xff79c6'
    cyan:    '0x8be9fd'
    white:   '0xbfbfbf'
  bright:
    black:   '0x4d4d4d'
    red:     '0xff6e67'
    green:   '0x5af78e'
    yellow:  '0xf4f99d'
    blue:    '0xcaa9fa'
    magenta: '0xff92d0'
    cyan:    '0x9aedfe'
    white:   '0xe6e6e6'
  dim:
    black:   '0x14151b'
    red:     '0xff2222'
    green:   '0x1ef956'
    yellow:  '0xebf85b'
    blue:    '0x4d5b86'
    magenta: '0xff46b0'
    cyan:    '0x59dffc'
    white:   '0xe6e6d1'
```



#  3. 参考资料

+ https://github.com/alacritty/alacritty

+ https://github.com/dracula/alacritty

  