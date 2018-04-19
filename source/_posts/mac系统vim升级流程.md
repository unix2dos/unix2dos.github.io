---
title: mac系统vim升级流程
date: 2016-11-17 17:49:00
tags: vim
---

#### 执行命令安装vim 注意要加上`--with-override-system-vi`
```bash
brew install vim --with-override-system-vi
```


#### 安装过程中如果出现 ruby.h找不到

执行下面的命令

```bash
cd /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/include/ruby-2.0.0/ruby
```

```bash
sudo ln -s ../universal-darwin15/ruby/config.h ./config.h
```
注意不要复制上面的, 对应自己的sdk版本,ruby版本,darwin版本


<!-- more -->
#### 给自己的vim做个别名, 注意自己vim的版本路径

```bash
alias vim="/usr/local/Cellar/vim/7.4.2152/bin/vim"
```



#### 升级vim后如果主题不显示
把自己主题文件放到这个 `/usr/local/Cellar/vim/7.4.2152/bin/vim`里


#### 让git也适应最新的vim

```bash
git config --global core.editor /usr/local/Cellar/vim/7.4.2152/bin/vim
```


--------

## 另外一种方案, 安装macvim

Install the latest version of MacVim. Yes, MacVim. And yes, the latest.

If you don't use the MacVim GUI, it is recommended to use the Vim binary that is inside the MacVim.app package (MacVim.app/Contents/MacOS/Vim). To ensure it works correctly copy the mvim script from the MacVim download to your local binary folder (for example /usr/local/bin/mvim) and then symlink it:



```
alias vim="/Applications/MacVim.app/Contents/MacOS/Vim"
git config --global core.editor /Applications/MacVim.app/Contents/MacOS/Vim
```

It requires Vim 7.3.885 or later with Lua support ("+lua").
```
brew install macvim --with-lua
```
