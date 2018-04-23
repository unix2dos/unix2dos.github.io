---
title: 'ssh_scp免密和服务器建立信任'
date: 2018-02-13 17:54:46
tags: ssh
---

+ 在mac上生成密钥
	
	生成两个文件`vultr`  `vultr.pub`
	
	`ssh-keygen -t rsa`   //passphrase可以为空

+ 发送到远程服务器

	第一种方式: 
	
	`scp ~/.ssh/vultr.pub root@207.246.80.69:/root/.ssh/authorized_keys`
	
	第二种方式:
	
	`ssh-copy-id -i ~/.ssh/vultr.pub root@207.246.80.69`

<!-- more -->
+ 添加到vultr的ssh key里(这一步可以不做)

	`https://my.vultr.com/sshkeys/`

	

+ 一键连接到ssh
	
	命令: `ssh -i ~/.ssh/vultr root@207.246.80.69`

+ scp files

	命令: `scp -i ~/.ssh/vultr files root@207.246.80.69:/root`
