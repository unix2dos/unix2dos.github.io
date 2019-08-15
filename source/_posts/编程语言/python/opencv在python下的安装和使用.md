---
title: opencv在python下的安装和使用
tags:
  - opencv
  - python
abbrlink: 415d206d
categories:
  - 编程语言
  - python
date: 2019-03-14 20:01:01
---



### 安装opencv


```shell
pip3 install numpy
pip3 install opencv-python
```

在安装`opencv-python`出现了以下错误信息:

```
 Could not fetch URL https://pypi.org/simple/opencv-python/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/opencv-python/ (Caused by SSLError(SSLError(1, u'[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:726)'),)) - skipping
```

解决方案: 

```shell
pip3 install --trusted-host pypi.org --trusted-host files.pythonhosted.org opencv-python
```



<!-- more -->

### opencv rotate code

```python
import sys
import cv2
import numpy as np


if len(sys.argv) != 2 :
    print("usage: ./images path")
    sys.exit(1)

img = cv2.imread(sys.argv[1])
cv2.imshow("Source", img)

img90 = np.rot90(img) # 旋转90
cv2.imshow("Rotate", img90)

cv2.waitKey(0)
cv2.destroyAllWindows()
```
