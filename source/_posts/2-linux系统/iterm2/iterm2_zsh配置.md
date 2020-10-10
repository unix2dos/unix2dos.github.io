---
title: iterm2配置zsh和常用插件
tags:
  - linux
  - iterm2
abbrlink: 6600d67c
categories:
  - 2-linux系统
  - iterm2
date: 2019-07-02 21:40:46
---

直接看效果图

![1](iterm2_zsh配置/3.png)

![1](iterm2_zsh配置/2.png)



<!-- more -->

# 1. 安装

### 1.1 安装 ohmyzsh

 https://github.com/ohmyzsh/ohmyzsh

注意安装会覆盖 `.zshrc`, 提前备份下

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

有效配置

```bash
 export ZSH="/Users/liuwei/.oh-my-zsh"
 ZSH_THEME="robbyrussell"
 plugins=(git)
 source $ZSH/oh-my-zsh.sh
```



### 1.2  安装主题 powerlevel10k

https://github.com/romkatv/powerlevel10k

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# 设置主题  ~/.zshrc
ZSH_THEME="powerlevel10k/powerlevel10k" 
```



+ 主题配置

```bash
POWERLEVEL9K_MODE='nerdfont-complete'
ZSH_THEME="powerlevel10k/powerlevel10k"
POWERLEVEL9K_CONTEXT_TEMPLATE='%n'
POWERLEVEL9K_CONTEXT_DEFAULT_FOREGROUND='white'
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX="%F{014}\u2570%F{cyan}\uF460%F{073}\uF460%F{109}\uF460%f "
POWERLEVEL9K_SHORTEN_DIR_LENGTH=1
POWERLEVEL9K_NODE_VERSION_BACKGROUND="002"
POWERLEVEL9K_NODE_VERSION_FOREGROUND="black"
POWERLEVEL9K_GO_VERSION_BACKGROUND="001"
POWERLEVEL9K_GO_VERSION_FOREGROUND="black"
POWERLEVEL9K_WIFI_BACKGROUND="003"
POWERLEVEL9K_WIFI_FOREGROUND="black"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon context ssh dir vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status proxy anaconda node_version go_version wifi)

# 看颜色
# for i in {0..255}; do print -Pn "%K{$i}  %k%F{$i}${(l:3::0:)i}%f " ${${(M)$((i%6)):#3}:+$'\n'}; done
```



### 1.3 安装字体

```bash
# 下载推荐字体
https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k
```

也可选择其他字体安装

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



# 2. 插件

### 2.1 git

```bash
#.zshrc
plugins=(git)
```

### 2.2 自动补充

https://github.com/zsh-users/zsh-autosuggestions

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

#.zshrc
plugins=(zsh-autosuggestions)
```

### 2.3 语法高亮

https://github.com/zsh-users/zsh-syntax-highlighting

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

#.zshrc
plugins=(zsh-syntax-highlighting)
```



# 3. 配色



### 3.1 配色项目

```bash
https://github.com/dracula/dracula-theme/ # 选用的这个!!!好看!!!支持各种终端
https://github.com/mbadolato/iTerm2-Color-Schemes # 这上面好多, 慢慢挑
https://github.com/MartinSeeler/iterm2-material-design 
```



### 3.2 安装dracula

https://github.com/dracula/iterm

```bash
git clone https://github.com/dracula/iterm.git

#Activating theme
iTerm2 > Preferences > Profiles > Colors Tab
Open the Color Presets... drop-down in the bottom right corner
Select Import... from the list

Select the Dracula.itermcolors file
Select the Dracula from Color Presets...
```



# 4. 遇到的问题

### 4.1 图案不显示

```bash
POWERLEVEL9K_MODE='nerdfont-complete' #这句话一定要在下面source之前
source $ZSH/oh-my-zsh.sh
```



# 5. 参考资料

+ [打造 Mac 下高颜值好用的终端环境](https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html)

+ https://medium.com/@Clovis_app/configuration-of-a-beautiful-efficient-terminal-and-prompt-on-osx-in-7-minutes-827c29391961


