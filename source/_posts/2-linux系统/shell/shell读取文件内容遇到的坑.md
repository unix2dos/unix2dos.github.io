---
title: shell读取文件内容遇到的坑
tags:
  - shell
  - linux
categories:
  - 2-linux系统
  - shell
abbrlink: '67723314'
date: 2020-04-14 15:54:46
---

文件内容如下, 需要把1234读取赋值给shell内部的变量nodeid

```
cat a.conf
nodeid 1234
```

<!-- more -->

shell解析文件如下

```shell
#!/bin/sh

nodeid=0


set_nodeid1(){
	cat a.conf | while read line
	do
		if [[ $line == nodeid* ]];
		then
			 nodeid=${line:7}
		fi
	done
}

set_nodeid2(){
	for line in `cat a.conf`
	do
		if [[ $line == nodeid* ]];
		then
			 nodeid=${line:7}
		fi
	done
}

set_nodeid3(){
	while read -r line
	do
		if [[ $line == nodeid* ]];
		then
			 nodeid=${line:7}
		fi
	done < a.conf
}


set_nodeid1
echo "set_nodeid1 is $nodeid"
set_nodeid2
echo "set_nodeid2 is $nodeid"
set_nodeid3
echo "set_nodeid3 is $nodeid"
```



输出内容如下:

```bash
set_nodeid1 is 0
set_nodeid2 is
set_nodeid3 is 1234
```



第一种方式创建了子shell, 赋值是子shell的, 没有影响到全局变量.

第二种方式, for循环的方式, 因为a.conf中间空格导致的,把一行循环了两次, 所以赋值了空

第三种方式得到正确的结果

