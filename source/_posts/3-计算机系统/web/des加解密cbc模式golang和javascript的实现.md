---
title: des加解密cbc模式golang和javascript的实现
tags:
  - golang
  - javascript
  - 加密
categories:
  - 3-计算机系统
  - web
abbrlink: 8b88eb3d
date: 2021-03-06 00:00:00
---

# 1. 介绍

### 1.1 对称加密算法

+ DES：DES 全称 Data Encryption Standard，是一种使用密钥加密的块算法。现在认为是一种不安全的加密算法，因为现在已经有用穷举法攻破 DES 密码的报道了。

+ 3DES（或称为 Triple DES）是三重数据加密算法（TDEA，Triple Data Encryption Algorithm）块密码的通称。它相当于是对每个数据块应用三次 DES 加密算法。由于计算机运算能力的增强，原版 DES 密码的密钥长度变得容易被暴力破解；3DES 即是设计用来提供一种相对简单的方法，即通过增加 DES 的密钥长度来避免类似的攻击，而不是设计一种全新的块密码算法。
+ AES 全称是 Advanced Encryption Standard，翻译过来是高级加密标准，它是用来替代之前的 DES 加密算法的。AES 加密算法的安全性要高于 DES 和 3DES，所以 AES 已经成为了主要的对称加密算法。+
+ 2000年代，DES逐渐被[3DES](https://zh.wikipedia.org/wiki/3DES)替代。2010年代，3DES逐渐被更安全的[高级加密标准](https://zh.wikipedia.org/wiki/高級加密標準)（AES）替代。

<!-- more -->

### 1.2 DES加密模式

##### 1.2.1 ECB

ECB模式又称电子密码本模式：Electronic codebook，是最简单的块密码加密模式，加密前根据加密块大小分成若干块，之后将每块使用相同的密钥单独加密，解密同理。

**优点：**

+ 简单；

+ 有利于并行计算；


+ 误差不会被传递；

**缺点：**

+ 不能隐藏明文的模式；

+ 可能对明文进行主动攻击

  

##### 1.2.2 CBC

密码分组链接（CBC，Cipher-block chaining）模式，由IBM于1976年发明，每个明文块先与前一个密文块进行异或后，再进行加密。在这种方法中，每个密文块都依赖于它前面的所有明文块。同时，为了保证每条消息的唯一性，在第一个块中需要使用初始化向量IV

**优点**：

+ 不容易主动攻击，安全性好于ECB，是SSL、IPSec的标准；

**缺点**：

+ 不利于并行计算；
+ 误差传递；
+ 需要初始化向量IV；

# 2. 实现

### 2.1 golang

```go
package main

import (
	"bytes"
	"crypto/des"
	"crypto/cipher"
	"encoding/base64"
	"fmt"
)

func DesEncryption(key, iv, plainText []byte) ([]byte, error) {
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}

	blockSize := block.BlockSize()
	origData := PKCS5Padding(plainText, blockSize)
	blockMode := cipher.NewCBCEncrypter(block, iv)
	cryted := make([]byte, len(origData))
	blockMode.CryptBlocks(cryted, origData)
	return cryted, nil
}

func DesDecryption(key, iv, cipherText []byte) ([]byte, error) {
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}

	blockMode := cipher.NewCBCDecrypter(block, iv)
	origData := make([]byte, len(cipherText))
	blockMode.CryptBlocks(origData, cipherText)
	origData = PKCS5UnPadding(origData)
	return origData, nil
}

func PKCS5Padding(src []byte, blockSize int) []byte {
	padding := blockSize - len(src)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(src, padtext...)
}

func PKCS5UnPadding(src []byte) []byte {
	length := len(src)
	unpadding := int(src[length-1])
	return src[:(length - unpadding)]
}


func main() {
	originalText := "Hello world"
	var key = "12345678"

	fmt.Println("加密前:",originalText)
	cryptoText,_ := DesEncryption([]byte(key), []byte(key), []byte(originalText))
	fmt.Println("加密后:", base64.StdEncoding.EncodeToString(cryptoText))
	decryptedText,_ := DesDecryption([]byte(key),[]byte(key), cryptoText)
	fmt.Println("解密后:", string(decryptedText))
}
```

运行后:

```ini
加密前: Hello world
加密后: SVd7Nf/Kw6itcZ02PHCmKQ==
解密后: Hello world
```



### 2.2 javascript

```html
<script src="https://cdn.bootcdn.net/ajax/libs/crypto-js/4.0.0/crypto-js.js"></script>

<script>
    var key = "12345678"

    function encryptByDESModeCBC(message) {
        var keyHex = CryptoJS.enc.Utf8.parse(key);
        var ivHex = CryptoJS.enc.Utf8.parse(key);
        encrypted = CryptoJS.DES.encrypt(message, keyHex, {
                iv:ivHex,
                mode: CryptoJS.mode.CBC,
                padding:CryptoJS.pad.Pkcs7
            }
        );
        return encrypted.ciphertext.toString(CryptoJS.enc.Base64);
    }

    function decryptByDESModeCBC(ciphertext) {
        var keyHex = CryptoJS.enc.Utf8.parse(key);
        var ivHex = CryptoJS.enc.Utf8.parse(key);
        var decrypted = CryptoJS.DES.decrypt({
            ciphertext: CryptoJS.enc.Base64.parse(ciphertext)
        }, keyHex, {
            iv: ivHex,
            mode: CryptoJS.mode.CBC,
            padding: CryptoJS.pad.Pkcs7
        });
        return decrypted.toString(CryptoJS.enc.Utf8);
    }

    var oriText = "Hello world"
    console.log("加密前:", oriText)
    var encText =  encryptByDESModeCBC(oriText)
    console.log("加密后:", encText)
    var decText =  decryptByDESModeCBC(encText)
    console.log("解密后:",decText)

</script>

```

运行后:

```ini
加密前: Hello world
加密后: SVd7Nf/Kw6itcZ02PHCmKQ==
解密后: Hello world
```



# 3. 参考资料

+ https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%8A%A0%E5%AF%86%E6%A8%99%E6%BA%96