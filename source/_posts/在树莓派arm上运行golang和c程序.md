---
title: "在树莓派arm上运行golang和c程序"
date: 2018-08-27 10:18:13
tags:
- golang
- arm
- linux
- 树莓派
---


## 树莓派基础设置

##### 树莓派修改键盘布局

```
sudo dpkg-reconfigure keyboard-configuration
```

选通用的101键PC键盘

在键盘layout选择中，选Other

然后在选项中，选English(US)

再选English(US, alternative international)

一直下一步,最后重启

```
sudo reboot
```



##### 树莓派修改启动进入终端界面

```

sudo raspi-config

boot option ->console

```
<!-- more -->


##### 树莓派开启ssh

```

sudo raspi-config

Select Interfacing Options

Navigate to and select SSH

Choose Yes

Select Ok

Choose Finish


sudo systemctl enable ssh

sudo systemctl start ssh
```



## 编译golang为arm可执行程序



```
GOOS=linux GOARCH=arm GOARM=6 go build -v
```



GOARM=5: use software floating point; when CPU doesn't have VFP co-processor

GOARM=6: use VFPv1 only; default if cross compiling; usually ARM11 or better cores (VFPv2 or better is also supported)

GOARM=7: use VFPv3; usually Cortex-A cores



## 编译c程序为arm可执行程序

##### 解决下载软件包连接不上的问题 connect to mirrors.opencas.cn....

   ```
sudo vim /etc/apt/sources.list 
   
替换源为
   
deb http://mirrors.shu.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi
   
sudo apt-get update&& sudo apt-get -y dist-upgrade&&sudo apt-get update 
   ```





## 其他系统交叉编译arm程序

##### 编译时指定cmake文件

```
cmake -DCMAKE_TOOLCHIAIN_FILE="/home/liuwei/transmission/cmake/arm.cmake" ..

make

make install DESTDIR=.
```

##### cmake样例:

```
# this one is important

SET(CMAKE_SYSTEM_NAME Linux)

set(CMAKE_SYSTEM_PROCESSOR arm)

#this one not so much

SET(CMAKE_SYSTEM_VERSION 1)

# specify the cross compiler

SET(CMAKE_C_COMPILER   arm-linux-gnu-gcc)

SET(CMAKE_CXX_COMPILER arm-linux-gnu-cpp)

# where is the target environment

# SET(CMAKE_FIND_ROOT_PATH  /my-path-to-toolchain/arm-unknown-linux-gnu)

# search for programs in the build host directories

SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# for libraries and headers in the target directories

SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)

SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

