---
title: mac下iterm2_zsh配置
tags: linux
abbrlink: 6600d67c
date: 2017-03-11 17:54:46
---


### 1. 安装zsh

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```



### 2. 修改主题

修改文件~/.zshrc中的ZSH_THEME一行，改成这个 

+ bullet-train

  ```
  ZSH_THEME="bullet-train" 
  
  https://github.com/caiogondim/bullet-train.zsh
  ```

+ powerlevel9k

  ```bash
  https://github.com/bhilburn/powerlevel9k/
  ```

  

<!-- more -->

### 3. 设置powerline

https://github.com/powerline/fonts 

把所有的都download 然后 执行 install.sh  执行了所有拷贝

+ 设置终端powerline字体

  然后在你的终端gui设置里面，把字体改成后缀为powerline的字体就行了, 我用的是: 18pt Meslo LG M DZ Regular for Powerline



### 4. zsh 插件

```bash
### autojump ###
brew install autojump
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
source $ZSH/oh-my-zsh.sh

### git ###
plugins=(git)
source ~/.oh-my-zsh/plugins/git/git.plugin.zsh

### 自动补充命令 ###
https://github.com/zsh-users/zsh-autosuggestions  

### 命令高亮 ###
https://github.com/zsh-users/zsh-syntax-highlighting
```

