---
title: android自动添加文件到android-mk
tags: android
abbrlink: 99b8fe68
categories:
  - 编程语言
  - android
date: 2016-10-18 16:53:11
---

将
```bash
LOCAL_SRC_FILES := hellocpp/main.cpp \  
                   ../../Classes/AppDelegate.cpp \  
                   ../../Classes/HelloWorldScene.cpp 
```
换成

```bash
FILE_LIST := hellocpp/main.cpp    
FILE_LIST += $(wildcard $(LOCAL_PATH)/../../Classes/*.cpp)    
LOCAL_SRC_FILES := $(FILE_LIST:$(LOCAL_PATH)/%=%)   
```

----

<!-- more -->
**另外一种方法:**

```bash
#遍历目录及子目录的函数  
define walk  
    $(wildcard $(1)) $(foreach e, $(wildcard $(1)/*), $(call walk, $(e)))  
endef  
  
#遍历Classes目录  
ALLFILES = $(call walk, $(LOCAL_PATH)/../../Classes)  
FILE_LIST := hellocpp/main.cpp  
#从所有文件中提取出所有.cpp文件  
FILE_LIST += $(filter %.cpp, $(ALLFILES))  
LOCAL_SRC_FILES := $(FILE_LIST:$(LOCAL_PATH)/%=%)
```
