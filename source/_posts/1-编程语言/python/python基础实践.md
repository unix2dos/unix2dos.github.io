---
title: python基础实践
tags:
  - python
categories:
  - 1-编程语言
  - python
abbrlink: d319c20d
date: 2019-08-27 23:10:46
---

### 1. 模块

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

<!-- more -->

### 2. python命名规范

##### 2.1 模块

- 模块尽量使用小写命名，首字母保持小写，尽量不要用下划线(除非多个单词，且数量不多的情况)

```python
# 正确的模块名
import decoder
import html_parser

# 不推荐的模块名
import Decoder
```

##### 2.2 类名

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

##### 2.3 函数

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

##### 2.4 变量名

- 变量名尽量小写, 如有多个单词，用下划线隔开

```python
if __name__ == '__main__':
    count = 0
    school_name = ''
```

##### 2.5 常量

- 常量使用以下划线分隔的大写命名

```python
MAX_CLIENT = 100
MAX_CONNECTION = 1000
CONNECTION_TIMEOUT = 600
```



### 3. python 方法返回多个值

```python
def f():
    return True, False
  
  
x, y = f()
print(x)
print(y)

gives:
True
False
```



### 4. sprintf()格式化输出

```python
# 字符串
'%s %s' % ('one', 'two')
'{} {}'.format('one', 'two')
one two


# int
'%d %d' % (1, 2)
'{} {}'.format(1, 2)
1 2


# float
'%f' % (3.141592653589793,)
'{:f}'.format(3.141592653589793)
3.141593


# 顺序
'{1} {0}'.format('one', 'two')
two one
```



### 5.  for循环

```python
# for in 循环
fruits = ["apple", "banana", "cherry"]
for x in fruits:
  print(x)
  
  
# range循环
for x in range(2, 6):
  print(x)#2 3 4 5
for x in range(2, 10, 3):
  print(x)# 2 5 8
  
  

# 循环带 k, v
presidents = ["Washington", "Adams", "Jefferson", "Madison", "Monroe", "Adams", "Jackson"]
for num, name in enumerate(presidents, start=1):
    print("President {}: {}".format(num, name))
    
  
# 循环多个
colors = ["red", "green", "blue", "purple"]
ratios = [0.2, 0.3, 0.1, 0.4]
for color, ratio in zip(colors, ratios):
    print("{}% {}".format(ratio * 100, color))
    
    
# 死循环
while True:
  pass
```



### 6. `if __name__ == 'main'`

一个python的文件有两种使用的方法，第一是直接作为脚本执行，第二是import到其他的python脚本中被调用（模块重用）执行。



`if __name__ == 'main'`: 的作用就是控制这两种情况执行代码的过程，在`if __name__ == 'main'`: 下的代码只有在第一种情况下（即文件作为脚本直接执行）才会被执行，而import到其他脚本中是不会被执行的。





### 7. `__pycache__`

在python中运行程序时，解释器首先将其编译为字节码，并将其存储在`__pycache__`文件夹。如果您在那里查找，您将发现一堆文件共享的名称。在项目文件夹中的Py文件，只有它们的扩展名才是其中之一.PYC或.pyo.。这些分别是字节码编译和优化字节码编译版本的程序的文件。


下次再执行工程时，若解释器发现这个 *.py 脚本没有修改过，就会跳过编译这一步，直接运行以前生成的保存在 __pycache__文件夹里的 *.pyc 文件。

这样工程较大时就可以大大缩短项目运行前的准备时间；如果你只需执行一个小工程，没关系 忽略这个文件夹就行。



### 8. 打开文件设置编码读取

```python
with open("1.html", "r", encoding='gbk') as f:
  contents = f.read()
  parse_html(contents)
```



### 9. 读写 json 文件

```python
class File(object):
    def __init__(self):
        self.name = "zk8.json"
        if not os.path.exists(self.name): # 没有就写一下
            with open(self.name, 'w'): pass


    def save(self, data):
        with open(self.name, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=4)


    def load(self):
        with open(self.name, 'r', encoding='utf-8') as f:
            data = json.load(f)
            return data

```



### 10. 数据结构 操作

```python
############ list ############
>>> fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
>>> fruits.count('apple')
2
>>> fruits.count('tangerine')
0
>>> fruits.index('banana')
3
>>> fruits.index('banana', 4)  # Find next banana starting a position 4
6
>>> fruits.reverse()
>>> fruits
['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange']
>>> fruits.append('grape')
>>> fruits
['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange', 'grape']
>>> fruits.sort()
>>> fruits
['apple', 'apple', 'banana', 'banana', 'grape', 'kiwi', 'orange', 'pear']
>>> fruits.pop()
'pear'


############ tuple ############

>>> t = 12345, 54321, 'hello!'
>>> t[0]
12345
>>> t
(12345, 54321, 'hello!')
>>> # Tuples may be nested:
... u = t, (1, 2, 3, 4, 5)
>>> u
((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))
>>> # Tuples are immutable:
... t[0] = 88888
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> # but they can contain mutable objects:
... v = ([1, 2, 3], [3, 2, 1])
>>> v
([1, 2, 3], [3, 2, 1])

############ set ############

>>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> print(basket)                      # show that duplicates have been removed
{'orange', 'banana', 'pear', 'apple'}
>>> 'orange' in basket                 # fast membership testing
True
>>> 'crabgrass' in basket
False

>>> # Demonstrate set operations on unique letters from two words
...
>>> a = set('abracadabra')
>>> b = set('alacazam')
>>> a                                  # unique letters in a
{'a', 'r', 'b', 'c', 'd'}
>>> a - b                              # letters in a but not in b
{'r', 'd', 'b'}
>>> a | b                              # letters in a or b or both
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
>>> a & b                              # letters in both a and b
{'a', 'c'}
>>> a ^ b                              # letters in a or b but not both
{'r', 'd', 'b', 'm', 'z', 'l'}




############ dict ############
>>> tel = {'jack': 4098, 'sape': 4139}
>>> tel['guido'] = 4127
>>> tel
{'jack': 4098, 'sape': 4139, 'guido': 4127}
>>> tel['jack']
4098
>>> del tel['sape']
>>> tel['irv'] = 4127
>>> tel
{'jack': 4098, 'guido': 4127, 'irv': 4127}
>>> list(tel)
['jack', 'guido', 'irv']
>>> sorted(tel)
['guido', 'irv', 'jack']
>>> 'guido' in tel
True
>>> 'jack' not in tel
False

# 避免 循环中删除 key 报错
for i in list(d):
  del d[i]
```





### 11. 类的特殊函数

```python
# __init__ 构造
class Foo:
    def __init__(self, a, b, c):
x = Foo(1, 2, 3) 

## __del__ 析构
class FileObject:
    def __del__(self):
        self.file.close()
        del self.file


# __call__ 类变成可调用
class Foo:
    def __call__(self, a, b, c):
x = Foo()
x(1, 2, 3) 


#__getattr__ 不存在的属性
class Dummy(object):
    def __getattr__(self, attr):
        return attr.upper()
d = Dummy()
d.does_not_exist # 'DOES_NOT_EXIST'
```



### 12. 枚举

```python
from enum import Enum 
class Animal(Enum):
    ant = 1
    bee = 2
    cat = 3
    dog = 4
```



### 13. 新的线程定时执行函数

```python
timer = threading.Timer(10, func)
timer.start()
```

