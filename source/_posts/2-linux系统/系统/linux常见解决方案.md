---
title: "linux常见解决方案"
date: 2021-01-18 00:00:00
tags:
- linux
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





