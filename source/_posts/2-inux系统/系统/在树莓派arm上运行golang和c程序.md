---
title: 在树莓派arm上运行golang和c程序
tags:
  - golang
  - arm
  - linux
  - 树莓派
abbrlink: 21dd607c
categories:
  - 2-inux系统
  - 系统
date: 2018-08-27 10:18:13
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



## 编译arm可执行程序

缺少的程序直接使用`包管理`下载即可

##### 解决下载软件包连接不上的问题 connect to mirrors.opencas.cn....

   ```
sudo vim /etc/apt/sources.list 
   
替换源为
   
deb http://mirrors.shu.edu.cn/raspbian/raspbian/ stretch main contrib non-free rpi
   
sudo apt-get update&& sudo apt-get -y dist-upgrade&&sudo apt-get update 
   ```
