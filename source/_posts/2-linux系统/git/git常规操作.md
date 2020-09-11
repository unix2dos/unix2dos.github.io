---
title: "git常规操作"
date: 2020-09-11 00:00:02
tags:
- git
- linux
---

# 1. revert

revert以新增一个 commit 的方式还原某一个 commit 的修改。这是一个安全的方法，因为它不会重写提交历史。

```bash
git revert <commit-id>
```

<!-- more -->

```bash
* d38ece4 - add 1 (25 minutes ago) <liuwei>

# 撤销上一次提交
git revert d38ece4


* edaa22e - Revert "add 1" (2 minutes ago) <liuwei>
* d38ece4 - add 1 (25 minutes ago) <liuwei>


# 我再撤上一次撤回的提交
git revert edaa22e

* 71903f8 - (HEAD -> cherry) Revert "Revert "add 1"" (50 seconds ago) <liuwei>
* edaa22e - Revert "add 1" (2 minutes ago) <liuwei>
* d38ece4 - add 1 (25 minutes ago) <liuwei>
```

如果为了commit干净, 可以使用`reset --hard +  push -f `(除非你知道你在干什么)



# 2. rebase

合并分支时候, merge 更安全, 保留的信息更多,  但是树结构会交错。rebase 会保证整个提交树特别整齐。

在你运行 git rebase 之前，一定要问问你自己「有没有别人正在这个分支上工作？」。如果答案是肯定的，那么把你的爪子放回去。

### 2.1  rebase介绍

rebase 算是git的一个黑魔法, 如果不会rebase, 别说会用git。

移步:  https://www.liuvv.com/p/b1718ace.html

### 2.2 git pull --rebase

当你在本地端的master分支开发时，你commit了一个小改变。同时其他人将他本周的修改推上了远地master分支。当你试着推上你的commit时，git会要求你先做git pull的动作。

当你下了git pull时，git会帮你产生一个`Merge remtoe-tracking branch ‘origin/master’` 的 commit。

虽然这没什么大不了的，但是这让log看起来有点混乱了。这个例子可以使用`git pull --rebase`。

这会先强制git抓取(pull)远地的修改，然后再重新接上(re-apply[rebase])你尚未推到远地端的commit，并且推上你的commit。这样一来git自动产生的Merge remote-tracking的log就不会出现了。



# 3. 其他

### 3.1 基础

移步:  https://www.liuvv.com/p/42eae4e7.html

### 3.2 cherrypick

移步: 

### 3.3 blame

查看某段代码是谁写的, blame 的意思为‘责怪’，你懂的。

```bash
git blame <file-name>
```

### 3.4 reflog(救命大招)

当骚操作出现问题的时候, 例如 reset, merge, rebase 中断了,  如果你找不到commit hash, 会不会奔溃?

每次更新了 HEAD 的 git 命令比如 commint、amend、cherry-pick、reset、revert 等都会被记录下来（不限分支），就像 shell 的 history 一样。 可以通过reflog查看。 

```bash
git reflog
```

### 3.5 查看分支文件

```bash
git show master:go.mod
```



# 4. 参考资料


+ https://github.com/521xueweihan/git-tips
+ https://github.com/geeeeeeeeek/git-recipes