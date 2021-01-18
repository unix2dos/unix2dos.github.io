---
title: "ssl证书后缀"
date: 2020-01-18 00:00:01
tags:
- ssl
---

# 1 PKCS

PKCS 全称是 Public-Key Cryptography Standards ，是由 RSA 实验室与其它安全系统开发商为促进公钥密码的发展而制订的一系列标准，PKCS 目前共发布过 15 个标准。 

<!-- more -->

常用的有：

### 1.1 PKCS#7 

Cryptographic Message Syntax Standard

PKCS#7 常用的后缀是： .P7B .P7C .SPC

### 1.2 PKCS#10 

Certification Request Standard

### 1.3 PKCS#12 

Personal Information Exchange Syntax Standard

PKCS#12 常用的后缀有： .P12 .PFX

### 1.4 X.509

X.509是常见通用的证书格式。所有的证书都符合为Public Key Infrastructure (PKI) 制定的 ITU-T X509 国际标准。

X.509 DER 编码(ASCII)的后缀是： .DER .CER .CRT

X.509 PAM 编码(Base64)的后缀是： .PEM .CER .CRT



# 2. 证书后缀

### 2.1 pkcs1-pkcs12

公钥加密(非对称加密)的一种标准(Pbulic Key Cryptography Standards),一般存储为`*.pn，*.p12`是包含证书和密钥的封装格式。

PKCS 全称是 Public-Key Cryptography Standards ，是由 RSA 实验室与其它安全系统开发商为促进公钥密码的发展而制订的一系列标准，PKCS 目前共发布过 15 个标准。

### 2.2 *.der

ASN.1是一套完整的数据结构与数据存储格式描述，BER/DER是ASN.1的二进制编码方式

Java和Windows服务器偏向于使用这种编码格式。

### 2.3 `*.cer *.crt`

两个指的都是证书。windows下叫cer，linux下叫crt；存储格式可以为pem也可以为der。.cer/.crt是用于存放证书，它是2进制形式存放的，不含私钥。

### 2.4 *.pem

Privacy Enhanced Mail，证书或密钥的Base64文本存储格式，可以单独存放证书或密钥，也可以同时存放证书或密钥。

打开看文本格式,以”—–BEGIN…”开头, “—–END…”结尾,内容是BASE64编码。

一般 Apache 和 Nginx 服务器应用偏向于使用 PEM 这种编码格式。

### 2.5 *.key

单独存放的pem格式的密钥，一般保存为*.key。

### 2.6 *.csr

证书签名请求(Certificate sign request)，包含证书持有人的信息，如国家，邮件，域名等。

### 2.7 *.pfx

微软iis的实现。用于存放个人证书/私钥，通常包含保护密码，2进制方式

### 2.8 *.jks

jks是Java密钥库(KeyStore)比较常见的一种格式。一般可用通过cer 或者pem 格式的证书以及私钥的进行转化为jks格式，有密码保护。所以它是带有私钥的证书文件，一般用户tomcat环境的安装。

### 2.9 *.crl

证书吊销列表(Certificate Revocation List)。



# 3. 参考资料:

+ https://www.codeleading.com/article/36513493623/