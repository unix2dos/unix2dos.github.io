---
title: git实用操作总结
tags: git
abbrlink: 42eae4e7
date: 2016-12-16 11:17:48
---

### git的配置
1. 安装git
2. 安装完成后，需要设置自己的用户名和email，在命令行输入：

```bash
git config --global user.name "levon"
git config --global user.email "levonfly@gmail.com"
```

### git和目录绑定
1. 在一个目录里可以通过git init命令把这个目录变成Git可以管理的仓库,然后通过以下命令绑定提交的地址

```bash
git remote add origin https://github.com/unix2dos/unix2dos.github.io
```

2. git clone 地址 就会创建目录和地址绑定
<!-- more -->

### git的基础操作
1. 命令git add 	      把文件添加到仓库
2. 命令git commit 	  把文件提交到仓库
3. 命令git pull 	  把远程仓库拉取文件
4. 命令git push       把文件提交到远程仓库
5. 命令git log 	      查看git提交日志
6. 如果嫌输出信息太多, 可以加上--pretty=oneline参数. 另外也可以花式log输出, git lg查看下

	```bash
	git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
	```
7. 命令git diff 查看版本之间文件修改变化

	```bash
	git diff 87b91b6 f9b3075 [--name-only]加上可以只看文件名字
	```

### git 回滚版本
在Git中，用HEAD表示当前版本,上一个版本就是HEAD^,上上一个版本就是HEAD^^,当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

1. 回滚到上一个版本
```bash
git reset --hard HEAD^
```
2. 回滚到任意一个版本
```bash
git reset --hard 版本号(通过git log查看)
```
3. 如果git回滚到历史版本后, git log只能看历史版本再以前的版本号, 不到未来的版本号怎么办?
>git 提供了一个命令git reflog用来记录你的每一次命令



### git回滚文件
+ 查看文件的修改记录

```bash
git log config.h
```
+ 查看文件版本的差别

```bash
git diff a3551 fd681 config.h
```
+  回退到指定的版本

```bash
git reset fd681 config.h
```
+  提交到本地参考

```bash
git commit -m "revert old file because commmit have a bug"   
```
+ 更新到工作目录

```bash
git checkout config.h   
```
+ 提交到远程仓库

```bash
git push origin master  
```


### git撤销操作

+ git修改文件后, 还没有add, commit.  这时撤销文件修改, 即回到上一版本的内容

```bash
git checkout -- file
```
命令中的--很重要，没有--，就变成了“创建一个新分支”的命令，我们在后面的分支管理中会再次遇到git checkout命令。

+ git add 文件后撤销add操作

```bash
git reset HEAD file
```

### git解决冲突

1. 建议手动解决冲突

2. 命令行可以使用别人或自己的版本

```bash
git checkout --theirs/--ours  file
```

### git全局配置

+ 中文不再显示8进制  git status显示中文

```bash
git config --global core.quotepath false
```

+ 设置代理, 因为国内一些原因下载的很慢

```bash
git config --global http.proxy 'localhost:8123'
git config --global --unset http.proxy
```

+ git区分文件大小写

git默认不区分文件大小写,导致文件名改了以后git状态没有改变,需要设置一下

```
git config core.ignorecase false
```


### git 分支操作
查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>


### git 标签操作

命令git tag <name>用于新建一个标签，默认为HEAD，也可以指定一个commit id；

git tag -a <tagname> -m "blablabla..."可以指定标签信息；

git tag -s <tagname> -m "blablabla..."可以用PGP签名标签；

命令git tag可以查看所有标签。

命令git push origin <tagname>可以推送一个本地标签；

命令git push origin --tags可以推送全部未推送过的本地标签；

命令git tag -d <tagname>可以删除一个本地标签；

命令git push origin :refs/tags/<tagname>可以删除一个远程标签。


### git 合并 commit

```
git rebase -i "合并前一个版本号"// 合并前一个 版本号

	pick 是用commit
	squash 是合并前一个

:wq 退出修改合并后的 commit log

git rebase --abort 如果出现失误来撤销
```


### github fork后更新源仓库的代码

```bash
git remote add upstream https://github.com/golang/go
git remote -v
git fetch upstream
git merge upstream/master
```

### git 增加 远程仓库 orgin(名字不一样)

```
git remote add github git@github.com:unix2dos/dht.git
git push github master
```

### git 分支修改名字

```
git branch -m 原名 新名
```


### git撤销操作

+ git push 后撤销

```
git revert <hash> 
```

+ commit消息撤销

```
git commit --amend -m '新的消息'
```
+ 回滚文件的改动(未有commit) 

```
git checkout -- <filename>
```
+ 回滚版本

```
git reset --hard <hash>  (--hard强制内容回归,如果修改内容保留不加此选项)
```

+ 停止追踪一个文件

你偶然把application.log加到代码库里了，现在每次你运行应用，Git都会报告在application.log里有未提交的修改。你把 *.log放到了.gitignore文件里，可文件还是在代码库里，你怎样才能让Git“撤销”对这个文件的追踪呢？

```
git rm --cached application.log
```


### git lg 完美显示

```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```