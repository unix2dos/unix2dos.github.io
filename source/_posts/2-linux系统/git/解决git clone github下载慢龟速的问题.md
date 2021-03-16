---
title: 解决git clone github下载慢龟速的问题
tags:
  - github
  - linux
categories:
  - 2-linux系统
  - git
abbrlink: 94e659ca
date: 2021-03-16 00:00:01
---

github 在家天天下载十几k, 忍你好久了, 实在是忍不了了..

<!-- more -->

# 1. 解决方案

### 1.1 代理(最实用)

复制 clashX 的终端代理命令, 复制一般如下: 

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

然而发现没有什么卵用, 还是10几 k. 

最后才发现我的 git clone 都是用的 ssh 协议,  虽然设置了 https 代理, 但 git clone git@github.com 没用。因为 ssh 不走 https!!!!

正确的方案是走 ssh 代理!!!

`vi ~/.ssh/config`

```bash
Host github.com
        HostName github.com
        IdentityFile ~/.ssh/github-unix2dos
        User unix2dos
        ProxyCommand nc -v -x 127.0.0.1:7890 %h %p
```



### 1.2 增强模式

https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public 

安装 clashx pro 版本。打开增强模式 就可以直接代理命令行了.

系统所有流量经过 clash，对软件透明.



### 1.3 减少拉取

如果 拉取 GitHub 项目仅仅是查看，可以加上 --depth=1 参数。

depth用于指定克隆深度，为1即表示只克隆最近一次commit.

```bash
git clone --depth 1 https://github.com/unix2dos/unix2dos.git
```

这种方法克隆的项目只包含最近的一次commit的一个分支，体积很小，即可解决项目过大导致Timeout的问题，但会产生另外一个问题，他只会把默认分支clone下来，其他远程分支并不在本地.

```bash
git clone -b ${branch} --depth=1 #可以拉对应分支
```



### 1.4 chrome 插件

 https://github.com/fhefh2015/Fast-GitHub

国内Github下载很慢，用上了这个插件后，下载速度嗖嗖嗖的~！

其实自己测试后发现, 也没有发现很快..



# 2. 参考资料

+ https://v2ex.com/t/730171
+ https://github.com/fhefh2015/Fast-GitHub

