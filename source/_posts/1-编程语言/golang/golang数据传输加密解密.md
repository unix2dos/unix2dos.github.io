---
title: golang数据传输加密解密
tags:
  - golang
  - gin
  - encrypt
categories:
  - 1-编程语言
  - golang
abbrlink: 8ba026f1
date: 2021-01-14 00:00:00
---

在前后端数据传输的过程中, 如果没有对数据加密, 抓包软件直接能看到我请求发的是什么数据，服务端给我返回的数据是什么。

并且可以用抓包软件修改响应数据返回给客户端，这样一来，客户端实际上接收到的数据并不是服务端给我的源数据，而是被第三者修改过的数据，如此一来，数据传输的安全就很有必要了。

<!-- more -->

解决方案可以用对称加密加密数据, 非对称加密加密key,   以客户端给服务端传输数据为例:

1. 服务端生成一对RSA秘钥，私钥放在服务端（不可泄露），公钥下发给客户端。
2. 客户端使用随机函数生成 key。
3. 客户端使用随机的 key 对传输的数据用AES进行加密。
4. 使用服务端给的公钥对 key进行加密。
5. 客户端将使用AES加密的数据  以及使用 RSA公钥加密的key  一起发给服务端。
6. 服务端拿到数据后，先使用私钥对加密的随机key进行解密，解密成功即可确定是客户端发来的数据，没有经过他人修改，然后使用解密成功的随机key对使用AES加密的数据进行解密，获取最终的数据。

这是单向的加密认证, 如果要实现双向加密验证, 就要生成两对公钥和私钥。



# 1. 步骤

### 1.1 生成rsa密钥对

生成私钥

```bash
openssl genrsa -out private_client.pem 1024
openssl genrsa -out private_server.pem 1024
```

生成公钥

```bash
openssl rsa -in private_client.pem -pubout -out public_client.pem
openssl rsa -in private_server.pem -pubout -out public_server.pem
```



### 1.2 加解密代码

+ rsa.go 非对称加密

```go
package encrypt

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"crypto/x509"
	"encoding/pem"
	"errors"
)

// BytesToPrivateKey bytes to private key
func BytesToPrivateKey(priv []byte) (*rsa.PrivateKey, error) {
	block, _ := pem.Decode(priv)
	enc := x509.IsEncryptedPEMBlock(block)
	b := block.Bytes
	var err error
	if enc {
		b, err = x509.DecryptPEMBlock(block, nil)
		if err != nil {
			return nil, err
		}
	}
	key, err := x509.ParsePKCS1PrivateKey(b)
	if err != nil {
		return nil, err
	}
	return key, nil
}

// BytesToPublicKey bytes to public key
func BytesToPublicKey(pub []byte) (*rsa.PublicKey, error) {
	block, _ := pem.Decode(pub)
	enc := x509.IsEncryptedPEMBlock(block)
	b := block.Bytes
	var err error
	if enc {
		b, err = x509.DecryptPEMBlock(block, nil)
		if err != nil {
			return nil, err
		}
	}
	ifc, err := x509.ParsePKIXPublicKey(b)
	if err != nil {
		return nil, err
	}
	key, ok := ifc.(*rsa.PublicKey)
	if !ok {
		return nil, errors.New("not ok")
	}
	return key, nil
}

// EncryptWithPublicKey encrypts data with public key
func EncryptWithPublicKey(msg []byte, pub *rsa.PublicKey) ([]byte, error) {
	hash := sha256.New()
	ciphertext, err := rsa.EncryptOAEP(hash, rand.Reader, pub, msg, nil)
	if err != nil {
		return nil, err
	}
	return ciphertext, nil
}

// DecryptWithPrivateKey decrypts data with private key
func DecryptWithPrivateKey(ciphertext []byte, priv *rsa.PrivateKey) ([]byte, error) {
	hash := sha256.New()
	plaintext, err := rsa.DecryptOAEP(hash, rand.Reader, priv, ciphertext, nil)
	if err != nil {
		return nil, err
	}
	return plaintext, nil
}
```

+ aes.go 对称加密

  ```go
  package encrypt
  
  import (
  	"bytes"
  	"crypto/aes"
  	"crypto/cipher"
  )
  
  // =================== CBC ======================
  func AesEncryptCBC(origData []byte, key []byte) (encrypted []byte) {
  	// 分组秘钥
  	// NewCipher该函数限制了输入k的长度必须为16, 24或者32
  	block, _ := aes.NewCipher(key)
  	blockSize := block.BlockSize()                              // 获取秘钥块的长度
  	origData = pkcs5Padding(origData, blockSize)                // 补全码
  	blockMode := cipher.NewCBCEncrypter(block, key[:blockSize]) // 加密模式
  	encrypted = make([]byte, len(origData))                     // 创建数组
  	blockMode.CryptBlocks(encrypted, origData)                  // 加密
  	return encrypted
  }
  func AesDecryptCBC(encrypted []byte, key []byte) (decrypted []byte) {
  	block, _ := aes.NewCipher(key)                              // 分组秘钥
  	blockSize := block.BlockSize()                              // 获取秘钥块的长度
  	blockMode := cipher.NewCBCDecrypter(block, key[:blockSize]) // 加密模式
  	decrypted = make([]byte, len(encrypted))                    // 创建数组
  	blockMode.CryptBlocks(decrypted, encrypted)                 // 解密
  	decrypted = pkcs5UnPadding(decrypted)                       // 去除补全码
  	return decrypted
  }
  func pkcs5Padding(ciphertext []byte, blockSize int) []byte {
  	padding := blockSize - len(ciphertext)%blockSize
  	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
  	return append(ciphertext, padtext...)
  }
  func pkcs5UnPadding(origData []byte) []byte {
  	length := len(origData)
  	unpadding := int(origData[length-1])
  	return origData[:(length - unpadding)]
  }
  
  // =================== ECB ======================
  func AesEncryptECB(origData []byte, key []byte) (encrypted []byte) {
  	newCipher, _ := aes.NewCipher(generateKey(key))
  	length := (len(origData) + aes.BlockSize) / aes.BlockSize
  	plain := make([]byte, length*aes.BlockSize)
  	copy(plain, origData)
  	pad := byte(len(plain) - len(origData))
  	for i := len(origData); i < len(plain); i++ {
  		plain[i] = pad
  	}
  	encrypted = make([]byte, len(plain))
  	// 分组分块加密
  	for bs, be := 0, newCipher.BlockSize(); bs <= len(origData); bs, be = bs+newCipher.BlockSize(), be+newCipher.BlockSize() {
  		newCipher.Encrypt(encrypted[bs:be], plain[bs:be])
  	}
  
  	return encrypted
  }
  func AesDecryptECB(encrypted []byte, key []byte) (decrypted []byte) {
  	newCipher, _ := aes.NewCipher(generateKey(key))
  	decrypted = make([]byte, len(encrypted))
  
  	for bs, be := 0, newCipher.BlockSize(); bs < len(encrypted); bs, be = bs+newCipher.BlockSize(), be+newCipher.BlockSize() {
  		newCipher.Decrypt(decrypted[bs:be], encrypted[bs:be])
  	}
  
  	trim := 0
  	if len(decrypted) > 0 {
  		trim = len(decrypted) - int(decrypted[len(decrypted)-1])
  	}
  
  	return decrypted[:trim]
  }
  func generateKey(key []byte) (genKey []byte) {
  	genKey = make([]byte, 16)
  	copy(genKey, key)
  	for i := 16; i < len(key); {
  		for j := 0; j < 16 && i < len(key); j, i = j+1, i+1 {
  			genKey[j] ^= key[i]
  		}
  	}
  	return genKey
  }
  ```

  

### 1.3 gin 中间件

```go
package middlewares

import (
	"bytes"
	"encoding/base64"
	"encoding/json"
	"io/ioutil"
	"math/rand"
	"net/http"
	"public/encrypt"

	"github.com/gin-gonic/gin"
)

type EncryptParam struct {
	Key           string `json:"key" form:"key"`
	EncryptedData string `json:"encrypted_data" form:"encrypted_data"`
}

type EncryptResponseWriter struct {
	gin.ResponseWriter
	Buff *bytes.Buffer
}

func (e *EncryptResponseWriter) Write(p []byte) (int, error) {
	return e.Buff.Write(p)
	//return e.ResponseWriter.Write(p) // 不再写底层的这个write
}

func Encrypt() gin.HandlerFunc {
	return func(c *gin.Context) {
		encryptType := c.Request.Header.Get("bb-encrypt")
		version := c.Request.Header.Get("bb-encrypt-ver")
		if encryptType == "" || encryptType == "none" {
			return
		}

		encryptWriter := &EncryptResponseWriter{c.Writer, bytes.NewBuffer(make([]byte, 0))}
		c.Writer = encryptWriter

		// 解密请求
		if encryptType == "request" || encryptType == "all" {

			param := EncryptParam{}
			if err := c.Bind(&param); err != nil {
				c.AbortWithStatus(http.StatusBadRequest)
				common.Log.Errorf("Bind EncryptParam err: %s", err)
				return
			}
			if param.Key == "" || param.EncryptedData == "" {
				c.AbortWithStatus(http.StatusBadRequest)
				common.Log.Error("EncryptedData is empty")
				return
			}

			cert, err := Dao.GetCert(version, "server")  // 此处是从数据库读取certs, 也可以本地读取文件
			if err != nil {
				c.AbortWithStatus(http.StatusBadRequest)
				common.Log.Errorf("GetCert err: %s", err)
				return
			}

			key, err := RsaDecryptData(cert.PrivateKey, param.Key)
			if err != nil {
				c.AbortWithStatus(http.StatusBadRequest)
				common.Log.Errorf("RsaDecryptData err: %s", err)
				return
			}
			data, err := AesDecryptData(key, param.EncryptedData)
			if err != nil {
				c.AbortWithStatus(http.StatusBadRequest)
				common.Log.Errorf("AesDecryptData err: %s", err)
				return
			}

			if c.Request.Method == http.MethodGet {
				c.Request.URL.RawQuery = data
			} else {
				c.Request.Body = ioutil.NopCloser(bytes.NewBuffer([]byte(data)))
			}

			common.Log.Infof("%v-middlewares-decrypt raw: %v", c.Request.URL.Path, data)
		}

		c.Next()

		normalReturn := func() {
			if _, err := encryptWriter.ResponseWriter.Write(encryptWriter.Buff.Bytes()); err != nil {
				common.Log.Error(err.Error())
			}
		}
		if encryptWriter.Status() != http.StatusOK { // 不成功, 直接返回
			normalReturn()
			return
		}

		encryptWriter.Header().Set("bb-encrypted", version)
		encryptWriter.Header().Set("bb-encrypt-ver", "0")
		// 加密返回
		if encryptType == "response" || encryptType == "all" {

			randomKey := RandStringRunes(16)
			cert, err := Dao.GetCert(version, "client") // 此处是从数据库读取certs, 也可以本地读取文件
			if err != nil {
				common.Log.Errorf("GetCert err: %s", err)
				return
			}

			key, err := RsaEncryptData(cert.PublicKey, randomKey)
			if err != nil {
				common.Log.Errorf("RsaEncryptData err: %s", err)
				return
			}
			encryptedData, err := AesEncryptData(randomKey, encryptWriter.Buff.String())
			if err != nil {
				common.Log.Errorf("AesEncryptData err: %s", err)
				return
			}

			data, err := json.Marshal(EncryptParam{Key: key, EncryptedData: encryptedData})
			if err != nil {
				common.Log.Error(err.Error())
			} else {
				common.Log.Infof("%v-middlewares-encrypt raw: %v", c.Request.URL.Path, encryptWriter.Buff.String())
				encryptWriter.Header().Set("bb-encrypt-ver", "1")
				if _, err := encryptWriter.ResponseWriter.Write(data); err != nil {
					common.Log.Error(err.Error())
				}
			}
		} else {
			normalReturn()
			return
		}
	}

}

// RSA加密
func RsaEncryptData(publicKey, data string) (res string, err error) {
	pk, err := encrypt.BytesToPublicKey([]byte(publicKey))
	if err != nil {
		return
	}

	eData, err := encrypt.EncryptWithPublicKey([]byte(data), pk)
	if err != nil {
		return
	}

	res = base64.StdEncoding.EncodeToString(eData)
	return
}

// RSA解密
func RsaDecryptData(privateKey, data string) (res string, err error) {
	eData, err := base64.StdEncoding.DecodeString(data)
	if err != nil {
		return
	}

	pk, err := encrypt.BytesToPrivateKey([]byte(privateKey))
	if err != nil {
		return
	}

	context, err := encrypt.DecryptWithPrivateKey(eData, pk)
	if err != nil {
		return
	}
	res = string(context)
	return
}

// AES加密
func AesEncryptData(key, data string) (res string, err error) {
	eData := encrypt.AesEncryptECB([]byte(data), []byte(key))
	res = base64.URLEncoding.EncodeToString(eData)
	return
}

// AES解密
func AesDecryptData(key, data string) (res string, err error) {
	eData, err := base64.URLEncoding.DecodeString(data)
	if err != nil {
		return
	}
	context := encrypt.AesDecryptECB(eData, []byte(key))
	res = string(context)
	return
}

var letterRunes = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")

// 随机字符串
func RandStringRunes(n int) string {
	b := make([]rune, n)
	for i := range b {
		b[i] = letterRunes[rand.Intn(len(letterRunes))]
	}
	return string(b)
}
```



# 2. 参考资料

+ https://blog.csdn.net/yuzhiqiang_1993/article/details/88641265
+ https://www.cnblogs.com/yjf512/p/10570922.html
+ https://blog.csdn.net/mirage003/article/details/87868999