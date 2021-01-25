---
title: linux常见解决方案
tags:
  - linux
categories:
  - 2-linux系统
  - 系统
abbrlink: 182cebe4
date: 2021-01-18 00:00:00
---

记录一些 linux 常见的问题解决方案.

<!-- more -->

### 1. 用户加到 sudo 用户组

```bash
vi /etc/sudoers

# 可以看到有个 sudo 用户组
# %sudo   ALL=(ALL:ALL) ALL
```

添加到 sudo组

```bash
usermod -aG sudo <username>
```

可以看到, sudo 组有这个用户了

```bash
vi /etc/group 
```



#### 2. 快速拷贝公钥到服务器

```bash
ssh-copy-id -i ~/.ssh/fhyx.pub   liuwei@49.234.15.70
```

