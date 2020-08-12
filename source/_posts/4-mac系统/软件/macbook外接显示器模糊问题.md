---
title: "macbook外接显示器模糊问题"
date: 2020-08-12 00:00:00
tags:
- mac
---

# 1. 解决方案

### 1.0 禁止SIP前提

`command + R` 进入mac安全模式, 禁止SIP

```bash
csrutil disable
```

<!-- more -->

### 1.1 TV 模式强制到 RGB mode

+ 参考教程:

  https://www.zhihu.com/question/19682094/answer/122193822

  

### 1.2 一键脚本开启HiDPI

HiDPI本质上是用软件的方式实现单位面积内的高密度像素。用四个像素点来表现一个像素，因此能够更加清晰细腻。

**高PPI(硬件) + HiDPI渲染(软件) = 更细腻的显示效果(retina)**

+ 项目地址

  https://github.com/xzhih/one-key-hidpi

+ 参考教程:

  https://blog.haitianhome.com/macbook-2k-hidpi.html

  选择时选择最大的即可



### 1.3 安装RDM软件

+ 项目地址

  https://github.com/avibrazil/RDM

+ 参考教程:

  https://www.zhihu.com/question/19682094/answer/669420795 



# 2. 显示器接口介绍

目前显示器的的接口有DP HDMI VGA DVI这几种，这些接口的形状也都不同，我们在选购主机的时候，一般都要考虑主机的显卡接口是否与显示器接口相匹配。VGA目前被淘汰掉了，这是因为VGA传输的是模拟信号。

从接口性能上来看，显示器接口的性能是 DP>HDMI>DVI>VGA。

+ **VGA接口**

  长的多针孔

  其中 VGA是模拟信号，目前已经被主流淘汰，这也是我们目前能在最新显卡上看到其他三种接口却看不到VGA接口的原因，DVI、HDMI、DP 都是数字信号，是目前的主流。

+ **DVI接口**

  DVI是高清接口，但不带音频，也就是说，DVI视频接线只传输画面图形信号，但不传输音频信号。但是DVI接口也有不少弊端：由于最初设计时是为PC端进行的，所以对于电视等兼容能力较差、只支持8bit的RGB信号传输、兼容性考虑，预留了不少引脚以支持模拟设备，造成接口体积较大。目前比较好的DVI接口能够传输2K画面，但也基本是极限了。

+ **HDMI接口**

  长的类似USB

  HDMI既能传输高清图形画面信号，也能够传输音频信号。

+ **DP接口**

  DP接口也是一种高清数字显示接口标准，可以连接电脑和显示器，也可以连接电脑和家庭影院。DP接口可以理解是HDMI的加强版，在音频和视频传输方面更加强悍。

  目前情况下，DP与HDMI在性能上没有多大区别。如果你使用3840*2160分辨率（4K），HDMI由于带宽不足，最大只能传送30帧，DP就没有问题。

+ 参考资料

  https://www.zhihu.com/question/19571221/answer/569037388



# 3. 参考资料

+ https://www.zhihu.com/question/19682094
+ https://sspai.com/post/57549
+ https://blog.haitianhome.com/macbook-2k-hidpi.html
+ https://zhuanlan.zhihu.com/p/43249762