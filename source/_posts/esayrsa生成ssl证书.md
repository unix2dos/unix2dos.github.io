---
title: esayrsa生成ssl证书
tags:
  - easyrsa
  - https
abbrlink: df25a0d8
date: 2018-07-07 12:01:26
---

### 下载release版本

https://github.com/OpenVPN/easy-rsa/releases

### 配置公钥基础设施变量

```
cp vars.example vars
vim vars
```

修改内容示例

```
set_var EASYRSA_REQ_COUNTRY "CN"
set_var EASYRSA_REQ_PROVINCE "BeiJing"
set_var EASYRSA_REQ_CITY "BeiJing"
set_var EASYRSA_REQ_ORG "Wise Innovation Inc."
set_var EASYRSA_REQ_EMAIL "user@mail.com"
set_var EASYRSA_REQ_OU "Wise Innovation"
```

<!-- more -->

### 初始化 easyrsa

1. 初始化

```
./easyrsa init-pki      # pki/{reqs,private} dir
```

2.  生成 crt


```
./easyrsa build-ca      # pki/private/ca.key pki/ca.crt
```

输入密码


Enter PEM pass phrase:


确认密码


Verifying - Enter PEM pass phrase:


输入 CA 的名称, 如: Wise Innovation CA


Common Name (eg: your user, host, or server name)[Easy-RSA CA]:



### 生成server证书 (因为用了通配符, 在 zsh 好像无效, 用 bash 执行命令)

```
./easyrsa build-server-full *.fhyx.online nopass  //用bash
```



   ### 生成client证书

```
./easyrsa build-client-full kc-spring-001 nopass 

./easyrsa build-client-full kc-box-001 nopass
```