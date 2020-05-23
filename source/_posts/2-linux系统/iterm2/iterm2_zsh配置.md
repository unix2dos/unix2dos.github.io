---
title: iterm2_zsh配置
tags:
  - linux
  - iterm2
abbrlink: 6600d67c
categories:
  - 2-linux系统
  - iterm2
date: 2019-07-02 21:40:46
---

先上下效果图

![1](iterm2_zsh配置/3.png)

![1](iterm2_zsh配置/2.png)



### 1. 配色iterm2

```bash
https://github.com/mbadolato/iTerm2-Color-Schemes # 这上面好多, 慢慢挑
https://github.com/dracula/dracula-theme/ # 选用的这个
https://github.com/MartinSeeler/iterm2-material-design 
```

<!-- more -->

### 2. 安装zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```



### 3. 修改主题 powerlevel9k

```bash
cd ~/.zgen/robbyrussell/oh-my-zsh-master/themes
git clone https://github.com/bhilburn/powerlevel9k/

.zshrc
ZSH_THEME="powerlevel9k/powerlevel9k"
```



### 4. 设置字体

```bash
# powerline
git clone https://github.com/powerline/fonts
./install.sh

# awesome-terminal-font
git clone https://github.com/gabrielelana/awesome-terminal-fonts
打开build文件夹双击安装

# nerd-fonts, 装这个即可
https://github.com/ryanoasis/nerd-fonts/

brew tap homebrew/cask-fonts
brew cask install font-hack-nerd-font
```



然后在iterm2里面，把字体改成后缀为powerline的字体就行了

![1](iterm2_zsh配置/1.png)





### 5. 主题配置

```bash
POWERLEVEL9K_MODE='nerdfont-complete'
ZSH_THEME="powerlevel9k/powerlevel9k"
POWERLEVEL9K_CONTEXT_TEMPLATE='%n'
POWERLEVEL9K_CONTEXT_DEFAULT_FOREGROUND='white'
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX="%F{014}\u2570%F{cyan}\uF460%F{073}\uF460%F{109}\uF460%f "
POWERLEVEL9K_SHORTEN_DIR_LENGTH=1
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon context battery dir vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status time dir_writable ip public_ip ram load background_jobs)
```



### 6. zsh 安装插件

```bash
#1. git
.zshrc
plugins=(git)

#2. autojump
brew install autojump #使用上就是 j xxx_Tab

.zshrc
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh


#3. 自动补充命令
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

.zshrc
plugins=(zsh-autosuggestions)

#4. 命令语法高亮
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

.zshrc
plugins=(zsh-syntax-highlighting)

#5. colors
gem install colorls

.zshrc
source $(dirname $(gem which colorls))/tab_complete.sh
```



### 7. 遇到的问题

- 图案不显示

```bash
POWERLEVEL9K_MODE='nerdfont-complete' #这句话一定要在下面source之前
source $ZSH/oh-my-zsh.sh
```



### 8. 参考资料

+ [打造 Mac 下高颜值好用的终端环境](https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html)

+ https://medium.com/@Clovis_app/configuration-of-a-beautiful-efficient-terminal-and-prompt-on-osx-in-7-minutes-827c29391961


