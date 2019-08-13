---
title: python使用正则后向引用替换字符串
tags: python
abbrlink: 136fd428
categories:
  - 编程语言
  - python
date: 2016-11-25 16:11:51
---
>工作需要把 `Mud makes my mom mad.` 这句话带有m的加上颜色,或者把某些单词加上颜色
>临时写了个脚本处理

```python
import re
import sys


# replace letter
#find = "m"
#str = "Mud makes my mom mad."

#replace key words
#find = "Mud|makes|mad"
#str  = "Mud makes my mom mad."

# how to use
# python b.py "m"	"Mud makes my mom mad." 
# python b.py "mud|mess|mop|make|the|help"	"Mud makes my mom mad."


find  = sys.argv[1] 
str   = sys.argv[2] 


result = re.sub(r'('+find+')', r'<color:#ff0000>\1</color>', str, 0, re.IGNORECASE)
print result

```
