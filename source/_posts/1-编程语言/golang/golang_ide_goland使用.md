---
title: golang_ide_goland使用
tags: golang
abbrlink: 5a4d0049
categories:
  - 1-编程语言
  - golang
date: 2018-05-10 18:17:48
---



# 1. 保存文件自动格式化和导入

`go fmt + go imports`

go to preferences ->Tools ->File Watchers and enable go fmt . This way on each save it will format the file.


goland tools->filewatchers->go fmt| go imports

<!-- more -->



# 2. 快捷键

+ 删除行 cmd + x

+ 复制行 cmd + d

+ 进入返回函数 cmd + [] 

  

# 3. 小技巧

### 3.1 sturct查看

+ 看定义所有的方法  cmd + b, 或者鼠标左键
+ 查看struct实现了哪些接口 cmd + u



### 3.2 interface查看

- 查看实现的Struct的列表    ctrl + h



### 3.3 小技巧

+ 呼出最近文件和常用功能 CMD + E , favorites 可以查看书签/断点/收藏

+ 文件导航  cmd + 7

  折叠, 展开,设置,隐藏

  按导出排序(建议), 按字母排序(建议),  私有函数(建议),  非当前文件的属性(不建议)



### 3.4 其他

+ 各种搜索 两次 shift

+ 比较文件 选择两个文件 cmd+d, 方便比较json

