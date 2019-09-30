---
title: github和gitee通过密钥来进行ssh连接
tags: git
abbrlink: a9407b5
categories:
  - 2-linux系统
  - git
date: 2017-04-01 17:54:46
---

## 一. github

#### 1 生成公钥私钥

```
ssh-keygen -t rsa -b 4096 -C "levonfly@gmail.com"
```

第一步sava file 写成github, 密码可以为空



<!-- more -->

#### 2 添加到github里面

```
cat github.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDmlu8zfJ+RuSREk0TGjuhujZbuuC1J+nAoQtkAmckfnbD8flJ6OEidSQbTqPRaQKIKObUKGobBeDWoHdJNVDGuyAPnSBnq7LI8ToKha91S4HLf8SNCtQHqCfMReNdPawav9rKN7hwos0Ho6fIMWtSRaJmZkw8gFwj4PuqfxuKzIm/hVaRCia8DJkLcyWfTYJAUkmoQHHIgJyNn0lTxs0AH0UzzfAoXiOzT6KRir5cKxhz+RCbz+ZTxmepDgM0uV/bN/rArJ/98QDknE8R4d5l88fXTR1vR98J7qgOrH+J6H15xtIInqtlGiHjv1FKu79p0t7o3WpajijiSsw8wFjlZ2Y4A8Hdm2+w7eWTUMasPbxWn4Jne2SAOWd4PsoOr3Fp0obH2RjQyibQ/WfGHHpOtJs9zGJoBs9YwmxexhhmHbCqGJ/KO6HYv9DssaLE9qG0gUshSiZtSbmaDOwttg2XfpieERrdt5SM4gsv7/MMQR5V2vZnzaKrh4++8oix48xAl27iR9qFXoqdOkXQ3CVXp15fMuQuhrzO73/mZnw8G0G5r5gzYt9ywwx+Jp9K1DLSrFESOLjAHec/8qbcSn7pUVkVkTUDvE8e+4bVnPRXe+MYa8aKybSx0OVB/foWKJlO5hgik/MHB3kVEQreoDJEv+ts5JIgEEsxtuEqfeDPdvw== levonfly@gmail.com
```
添加到  https://github.com/settings/keys

+ ssh-rsa 要复制
+ 邮箱不复制
+ 生成的密钥只能用在一个帐号上面




#### 3 测试是否连接到github, 现在带上私钥

```
ssh -T git@github.com -i github
```


#### 4 添加到config, git使用私钥

```
host github.com
 HostName github.com
 IdentityFile ~/.ssh/github
 User levon
```



## 二. 码云

#### 0 添加的时候一定要添加ssh地址

```
git remote add liuwei git@gitee.com:metrics-client-res/ReadingMate.git
```

#### 1 生成公钥私钥

```
ssh-keygen -t rsa -C "levonfly@gmail.com"  
```
第一步sava file 写成mayun, 密码可以为空

#### 2 添加到gitee里面

```
cat ~/.ssh/mayun.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDOD++FY3wmtogXUNkSVl7ZLF8jLFJsua79Tvg5ywY+YngvjCW7EsWps7M3MeVYBxht4vuFrA3qeDD3UlTE8hKJUHJaCSrfjWT/uTo9OQKK2hW/nyvNJomz5aqBoArEIPD5Ab6cqpOpMElrUDEKrfW+3FWR++mpS/ig9NNR5l2GuIYIJOt4NOkXPALd8gWjRMPedOI8MJLstK4M7BinMaoSgwOoNWYrEmDXDcJpNt7c40T83npGd5TLfN3Oq50aZwSPfzBJfDzk+kBdplrg+a7YR50TP9/URE4MKrmRToOXyVuCucRn6WTskVbt+lJqBnzO/CkTRvIeOCuaZcQqLoTH levonfly@gmail.com
```
添加到地址 https://gitee.com/profile/sshkeys

#### 3 测试是否连接到码云, 现在带上私钥

```
ssh -T git@gitee.com -i mayun
```

#### 4 添加到config, git使用私钥

```
host gitee.com
 HostName gitee.com
 IdentityFile ~/.ssh/mayun
 User levon
```
