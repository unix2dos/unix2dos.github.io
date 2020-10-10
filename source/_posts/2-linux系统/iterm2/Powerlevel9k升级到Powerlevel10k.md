---
title: Zsh主题Powerlevel9k升级到Powerlevel10k
tags:
  - zsh
  - iterm2
categories:
  - 2-linux系统
  - iterm2
abbrlink: 4fd520f9
date: 2020-05-23 12:23:01
---



为什么升级？

Powerlevel9k项目不再维护，Powerlevel10k更快更强大（10-100倍的性能提升）。

Powerlevel10k并且完美兼容Powerlevel9k, 以前的配置参数可以不用任何修改.

<!-- more -->

### 1. 替换

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k

# Replace ZSH_THEME="powerlevel9k/powerlevel9k" with ZSH_THEME="powerlevel10k/powerlevel10k".
```



### 2. 配置

可以通过 `p10k configure` 安装推荐字体. 也可以`p10k configure`进行傻瓜式主题配置. 不过还是建议自己定制.

```
p10k configure
```



### 3. 我的配置

```yaml
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



### 4. 参考资料

+ https://github.com/romkatv/powerlevel10k
+ https://www.liuvv.com/p/6600d67c.html

