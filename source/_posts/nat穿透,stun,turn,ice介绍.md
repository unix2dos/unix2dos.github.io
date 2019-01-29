---
title: "nat穿透,stun,turn,ice介绍"
date: 2019-01-29 15:54:16
tags:
- nat
---

### 1. NAT网络地址转换:  

资料: https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2



完全圆锥型NAT( Full Cone NAT )

地址限制圆锥型NAT( Address Restricted Cone NAT )

端口限制圆锥型NAT( Port Restricted Cone NAT ) 

对称型NAT( Symmetric NAT)

<!-- more -->

### 2. STUN(Session Traversal Utilities for NAT)

资料: <https://zh.wikipedia.org/wiki/STUN>



STUN，NAT会话穿越应用程序 <https://tools.ietf.org/html/rfc5389>）是一种[网络协议](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE)，它允许位于[NAT](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2)（或多重NAT）后的客户端找出自己的公网地址，查出自己位于哪种类型的NAT之后以及NAT为某一个本地端口所绑定的Internet端端口。这些信息被用来在两个同时处于NAT路由器之后的主机之间创建UDP通信。该协议由RFC 5389定义。

四种主要类型中有三种是可以使用的：[完全圆锥型NAT](https://zh.wikipedia.org/w/index.php?title=%E5%AE%8C%E5%85%A8%E5%9C%86%E9%94%A5%E5%9E%8BNAT&action=edit&redlink=1)、[受限圆锥型NAT](https://zh.wikipedia.org/w/index.php?title=%E5%8F%97%E9%99%90%E5%9C%86%E9%94%A5%E5%9E%8BNAT&action=edit&redlink=1)和[端口受限圆锥型NAT](https://zh.wikipedia.org/w/index.php?title=%E7%AB%AF%E5%8F%A3%E5%8F%97%E9%99%90%E5%9C%86%E9%94%A5%E5%9E%8BNAT&action=edit&redlink=1)——但大型公司网络中经常采用的对称型NAT（又称为双向NAT）则不能使用。



### 3. TURN(Traversal Using Relay NAT)

资料:  <https://zh.wikipedia.org/wiki/TURN>



TURN（ <https://tools.ietf.org/html/rfc5766>），是一种数据传输协议（data-transfer protocol）。允许在TCP或UDP的连在线跨越[NAT](https://zh.wikipedia.org/wiki/NAT)或[防火墙](https://zh.wikipedia.org/wiki/%E9%98%B2%E7%81%AB%E5%A2%99)。



### 4. ICE(Interactive Connectivity Establishment) 

资料: https://zh.wikipedia.org/wiki/%E4%BA%92%E5%8B%95%E5%BC%8F%E9%80%A3%E6%8E%A5%E5%BB%BA%E7%AB%8B>



ice是一种综合性的[NAT穿越](https://zh.wikipedia.org/wiki/NAT%E7%A9%BF%E8%B6%8A)的技术, 是由[IETF](https://zh.wikipedia.org/wiki/IETF)的MMUSIC工作组开发出来的一种framework，可集成各种[NAT穿透](https://zh.wikipedia.org/wiki/NAT%E7%A9%BF%E9%80%8F)技术，如[STUN](https://zh.wikipedia.org/wiki/STUN)、[TURN](https://zh.wikipedia.org/wiki/TURN)（Traversal Using Relay NAT，中继NAT实现的穿透）、RSIP（Realm Specific IP，特定域IP）等。该framework可以让SIP的客户端利用各种NAT穿透方式打穿远程的[防火墙](https://zh.wikipedia.org/wiki/%E9%98%B2%E7%81%AB%E5%A2%99)。



### 5. 三个协议介绍

资料:  <https://blog.csdn.net/byxdaz/article/details/52786600>  
