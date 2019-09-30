---
title: github多帐号登录的问题
tags:
  - github
  - git
categories:
  - 2-linux系统
  - git
abbrlink: 899d7696
date: 2019-09-30 17:22:46
---



### 1. 同一台电脑有2个github账号？

+ 首先要为每个帐号生成公钥私钥对, 并且设置到 github 里, 参考 {% post_link 2-linux系统/git/github和gitee通过密钥来进行ssh连接 %}

+ 修改 `~/.ssh/config`, 设置如下

```bash
Host unix2dos
        HostName github.com
        IdentityFile ~/.ssh/github-unix2dos
        User unix2dos
Host levonfly
        HostName github.com
        IdentityFile ~/.ssh/github-levonfly
        User levonfly
```

测试:
```bash
ssh -T git@unix2dos
ssh -T git@levonfly
```
<!-- more -->

+ 要修改仓库的 remote url, 对应 `~/.ssh/config` 所填写的值.  注意, 要修改 git@后面的这个值

```bash
git remote set-url origin git@unix2dos:unix2dos/LevonRecord.git
```



- 一定要设置用户名和邮箱.否则虽然可以提交 commit, 但是不认识你是谁

  

- 建议 global 用一个,  其他项目用另外的用户名

```bash
git config -l

git config --global user.name "unix2dos"
git config --global user.email "levonfly@gmail.com" 


git config user.name "levonfly"
git config user.email "6241425@qq.com" 
```



### 2. mac切换用户提交失败的问题

提交总是出现permission denied的问题，用git config --global更新了username和email也不行。

mac os原因是即便更新了username和email，mac在git push时还是会使用历史账号的密码。

> 解决方法如下：

1. 进入Keychain Access (不知道在哪儿的可以command+space查找)
2. 在搜索框输入'git'进行查找，将找到的文件删掉，这里保存了历史账号的信息
3. 删除之后重新用git config --global更新username和email即可，之后git push会要求你输入username和password
4. done!


参考: https://www.zhihu.com/question/23028445/answer/399033488




### 3. 参考资料

+ https://gist.github.com/jexchan/2351996
+ https://www.zhihu.com/question/23028445/answer/399033488