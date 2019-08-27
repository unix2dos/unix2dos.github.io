---
title: shell批量文件内容复制到一个文件内
tags: shell
abbrlink: 875a198
categories:
  - 2-linux系统
  - shell
date: 2016-11-29 17:54:46
---

> 公司需要把所有代码放到一个文件内,加上版权信息. 于是用shell简单的处理了下

```shell
#!/bin/sh

NAME="a.txt"
if [ -f $NAME ]; then
	`rm $NAME`
fi

DIR=""
FILE=""
for file in `ls -R`
do
	if [ -f $file ]; then
		if [ $file = "a.sh" ];then
			continue	
		fi
#		echo "===================== $file begin =====================" >> $NAME
#		`cat $file >> $NAME`
#		echo "===================== $file end =====================" >> $NAME
		echo $file		
	else 
		if  [ ${file:0:1} = "." ];then
			DIR=${file/://}
		else  
			if [ "$DIR" != "" ] && [ ${DIR:0:6} = "./base" ];then
				continue #此处可以过滤不想要的文件夹	
			fi
			FILE=$DIR$file
			if [ -f $FILE ]; then
		#		echo "===================== $file begin =====================" >> $NAME
		#		`cat $FILE >> $NAME`
		#		echo "===================== $file end =====================" >> $NAME
				echo $FILE
			fi
		fi
	fi
done
```
