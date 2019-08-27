---
title: python_requests的使用
tags:
  - requests
  - python
categories:
  - 1-编程语言
  - python
abbrlink: 4a761254
date: 2019-08-31 10:35:00
---



request是一个简答优雅的python HTTP库，相较于python标准库中的urllib和urllib2的库，requests更加的便于理解和使用.



### 1. 安装 requests

```bash
pip install requests
```

<!-- more -->

### 2. requests包的使用

##### get

```
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get('https://httpbin.org/get', params=payload)

>>> print(r.url)
https://httpbin.org/get?key2=value2&key1=value1
```



##### post

```python
import requests
import json

headers = {'content-type': 'application/json'}
url = 'http://192.168.3.45:8080/api/v2/event/log'

data = {"eventType": "AAS_PORTAL_START", "data": {"uid": "hfe3hf45huf33545", "aid": "1", "vid": "1"}}
params = {'sessionKey': '9ebbd0b25760557393a43064a92bae539d962103', 'format': 'xml', 'platformId': 1}

requests.post(url, params=params, data=json.dumps(data), headers=headers, timeout=2)
```



##### parse 

```python
import requests
requests.get(url).json()
```





### 3. 参考资料:

+ https://2.python-requests.org//zh_CN/latest/user/quickstart.html