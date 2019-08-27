---
title: "python基础实践"
date: 2019-08-27 23:10:46
tags:
- python
---


### 1. `if __name__ == '__main__'` 



一个python的文件有两种使用的方法，第一是直接作为脚本执行，第二是import到其他的python脚本中被调用（模块重用）执行。因此if `__name__ == 'main'`: 的作用就是控制这两种情况执行代码的过程，在`if __name__ == 'main'`: 下的代码只有在第一种情况下（即文件作为脚本直接执行）才会被执行，而import到其他脚本中是不会被执行的。

<!-- more -->



### 2. 模块

Python 模块(Module)，是一个 Python 文件，以 .py 结尾，包含了 Python 对象定义和Python语句。

```python
from pkg.func import hello
# pkg 是模块名字,就是目录名字
# pkg.func 是 pkg 目录下的 func 文件
# hello 是 func 文件的 hello函数


from pkg.topic import Topic
# Topic 是 topic 文件的 类
```



+ `__init__.py` ,如果目录中存在该文件，该目录就会被识别为 module package 。
+ `__init__.py` 在包被导入时会被执行。该文件就是一个正常的python代码文件，因此可以将初始化代码放入该文件中。



### 3. pycharm unresolved reference

+ 在项目的文件夹(module 的上一级)右键 `Mark Directory as` -> `Source root`

+ `File` -> `Invalidate Caches / Restart` and restart PyCharm.



### 4. python命名规范

##### 4.1 模块

- 模块尽量使用小写命名，首字母保持小写，尽量不要用下划线(除非多个单词，且数量不多的情况)

```python
# 正确的模块名
import decoder
import html_parser

# 不推荐的模块名
import Decoder
```

##### 4.2 类名

- 类名使用驼峰(CamelCase)命名风格，首字母大写，私有类可用一个下划线开头

```Python
class Farm():
    pass

class AnimalFarm(Farm):
    pass

class _PrivateFarm(Farm):
    pass
```

- 将相关的类和顶级函数放在同一个模块里. 不像Java, 没必要限制一个类一个模块.

##### 4.3 函数

- 函数名一律小写，如有多个单词，用下划线隔开

```python
def run():
    pass

def run_with_env():
    pass
```

- 私有函数在函数前加一个下划线_

```python
class Person():
    def _private_func():
        pass
```

##### 4.4 变量名

- 变量名尽量小写, 如有多个单词，用下划线隔开

```python
if __name__ == '__main__':
    count = 0
    school_name = ''
```

##### 4.5 常量

- 常量使用以下划线分隔的大写命名

```python
MAX_CLIENT = 100
MAX_CONNECTION = 1000
CONNECTION_TIMEOUT = 600
```