---
title: python读取文件去除html_xml标签
tags: ["python"]
abbrlink: f3ea3847
categories:
  - 1-编程语言
  - python
date: 2016-10-19 15:53:58
---

> 自己写的一个简单去除html标签(xml)的脚本
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<text font="Palatino Linotype">
<p>
<s end_audio="262" start_audio="13">
<w end_audio="94" id="0" start_audio="13" variants="mud">Mud</w>
<w end_audio="122" id="1" start_audio="95">makes</w>
<w end_audio="140" id="2" start_audio="123">my</w>
<w end_audio="215" id="3" start_audio="141">mom</w>
<w end_audio="262" id="4" start_audio="216" variants="mad">mad.</w>
</s>
</p>
</text>
```

```python
import re
import sys

# how to use
# pythone a.py C6M01B1-001.xml

file = sys.argv[1]
f = open(file, 'r')

res = ""
for line in f.readlines():
	#str = re.sub(r'</?\w+[^>]*>','',line)
	str = re.sub(r'<(/|\?)?\w+[^>]*>','',line)
	if str != '\r\n':
	res = res + str
	res = re.sub('\r\n',' ',res)
	print res
#print "len =",len(res)
```
