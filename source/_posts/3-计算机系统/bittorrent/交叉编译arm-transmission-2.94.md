---
title: 交叉编译arm-transmission-2.94
tags:
  - arm
  - linux
  - transmission
  - bt
abbrlink: 73da9612
categories:
  - 3-计算机系统
  - bittorrent
date: 2018-10-08 17:02:00
---



# 1. 编译平台准备工作

1. 下载arm-none-linux-gnueabi-gcc

2. 下载transmission-2.94

3. 新建ARM文件夹

4. 解压arm-none-linux-gnueabi-gcc和transmission-2.94到ARM文件夹

5. 设置编译平台环境变量

   ```shell
   export PATH="/root/ARM/external-toolchain/bin:$PATH"
   export cross=arm-none-linux-gnueabi-
   export CC="${cross}gcc"
   ```

6. 编译的时候一定要注意看log, 是arm-none-linux-gnueabi-gcc编译的才是正确的

<!-- more -->



# 2. 目标平台开始编译

### 2.1 transmission

```shell
./configure --host="arm-none-linux-gnueabi" --prefix=/usr/local --without-gtk --without-systemd_daemon  --disable-mac --enable-utp --disable-nls  --enable-utp --enable-lightweight --disable-cli --enable-daemon  PKG_CONFIG="/usr/bin/pkg-config" PKG_CONFIG_PATH="/root/ARM/opt/lib/pkgconfig"

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install DESTDIR=/root/ARM/transmission-2.94/a(绝对路径)
```



git clone下来的需要执行./autogen.sh

```shell
./autogen.sh --host="arm-none-linux-gnueabi" --prefix=/usr/local --without-gtk --without-systemd  --disable-mac --enable-utp --disable-nls  --enable-utp --enable-lightweight --disable-cli --enable-daemon  PKG_CONFIG="/usr/bin/pkg-config" PKG_CONFIG_PATH="/root/ARM/opt/lib/pkgconfig" 

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install DESTDIR=/root/transmission/a(绝对路径)
```



错误1. 出现No package 'libevent' found ->安装libevent

错误2. fatal error: curl/curl.h: No such file or directory -> 安装curl

错误3. rpcimpl.c:16:18: fatal error: zlib.h: No such file or directory ->安装zlib

错误4. fatal error: systemd/sd-daemon.h: No such file or directory ->需要安装systemd, 此处强烈建议使用`--without-systemd_daemon`选项, 否则编译systemd又是一堆依赖



### 2.2 libevent

```shell
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
cd libevent-2.0.21-stable
./configure --host="arm-none-linux-gnueabi" --prefix="/root/ARM/opt/"

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install
```



### 2.3 libcurl

```shell
wget https://curl.haxx.se/download/curl-7.61.1.tar.gz
tar zxvf curl-7.61.1.tar.gz
cd curl-7.61.1
./configure --prefix="/root/ARM/opt/" --target=arm-none-linux-gnueabi --host=arm-none-linux-gnueabi --with-zlib="/root/ARM/opt" --with-ssl="/root/ARM/opt"

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install
```



错误1. configure: error: /root/ARM/opt is a bad --with-ssl prefix! -> 安装openssl



### 2.4 openssl

```shell
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
tar zxvf openssl-1.1.1.tar.gz
cd openssl-1.1.1
./Configure linux-generic32 shared  -DL_ENDIAN --prefix=/root/ARM/opt --openssldir=/root/ARM/opt

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld" MAKEDEPPROG="${cross}gcc" PROCESSOR=ARM
make install
```



### 2.5 zlib

```shell
wget http://zlib.net/zlib-1.2.11.tar.gz
tar zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure --prefix="/root/ARM/opt"

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install
```



# 3. 编译systemd

参考: https://wiki.beyondlogic.org/index.php?title=Cross_Compiling_SystemD_for_ARM

#### 1. libkmod

```shell
wget https://www.kernel.org/pub/linux/utils/kernel/kmod/kmod-17.tar.gz
./configure --host=arm-none-linux-gnueabi --prefix=/root/ARM/opt/ 

make 
make install 
```

#### 2. libffi

```shell
wget https://sourceware.org/ftp/libffi/libffi-3.2.1.tar.gz
./configure --prefix=/root/ARM/opt/  CC=arm-none-linux-gnueabi-gcc --host=arm-none-linux-gnueabi   

make
make install
```

#### 3. pcre

```shell
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.42.tar.gz
./configure --prefix=/root/ARM/opt/ --host=arm-none-linux-gnueabi   CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib"

make
make install
```

#### 4. libattr

```shell
wget https://download-mirror.savannah.gnu.org/releases/attr/attr-2.4.48.tar.gz
./configure --host=arm-none-linux-gnueabi --prefix=/root/ARM/opt/

make CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld" 
make install
```

#### 5. libcap

```shell
wget https://www.kernel.org/pub/linux/libs/security/linux-privs/libcap2/libcap-2.24.tar.xz 
make prefix=/root/ARM/opt/  BUILD_CC=gcc  CC="${cross}gcc" AR="${cross}ar" RANLIB="${cross}ranlib" LD="${cross}ld"
make install
```

make不成功修改文件:

```shell
vi libcap/cap_file.c

-#ifdef VFS_CAP_U32
+#if defined (VFS_CAP_U32) && defined (XATTR_NAME_CAPS)
```

#### 6. glibc(没有成功....)

```shell
wget http://ftp.gnome.org/pub/gnome/sources/glib/2.52/glib-2.52.3.tar.xz

export glib_cv_stack_grows=no; \
export glib_cv_uscore=no; \
export ac_cv_func_posix_getpwuid_r=no; \
export ac_cv_func_posix_getgrgid_r=no; \
CFLAGS=-I/root/ARM/opt/include \
LDFLAGS=-L/root/ARM/opt/lib \

 
./configure --host=arm-none-linux-gnueabi --prefix=/root/ARM/opt/ --disable-libmount
```



# 4. 目标平台运行

+ systemd编译失败, 可在编译transmission的时候去掉systemd

+ 目标平台执行transmission的环境变量(可忽略)

```shell
export PATH=/dev/opt/bin:$PATH
export LD_LIBRARY_PATH=/dev/opt/lib:$LD_LIBRARY_PATH
export CURL_CA_BUNDLE=/mnt/Sync2/ca.crt
export TR_CURL_SSL_CERT=/mnt/Sync2/cert.pem
export TR_CURL_SSL_CERT=/mnt/Sync2/cert.pem
export TR_CURL_SSL_KEY=/mnt/Sync2/key.pem
export STNOUPGRADE=1
```



# 5. 参考资料

+ https://wiki.beyondlogic.org/index.php?title=Cross_Compiling_SystemD_for_ARM