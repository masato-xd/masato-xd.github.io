---
title: python中sort和sorted的使用
date: 2017-07-06 22:03:30
tags: python
categories: python
---

我们需要对List进行排序，Python提供了两个方法
- **List的成员函数 sort 进行排序**
- **built-in函数 sorted 进行排序**

<!-- more -->

两者的区别
```python
>>> help(sorted)
Help on built-in function sorted in module __builtin__:

sorted(...)
    sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list

>>> help(list.sort)
Help on method_descriptor:

sort(...)
    L.sort(cmp=None, key=None, reverse=False) -- stable sort *IN PLACE*;
    cmp(x, y) -> -1, 0, 1

```
> **iterable:** 可迭代对象
> **cmp:** 用于比较的函数，比较什么由key决定
> **key: ** 列表元素的某个属性和函数进行作为关键字
> **reverse: ** 排序规则. reverse = True, 默认为False, True是代表倒序

**简单举几个栗子:**
```python
>>> a = [999, 33, 66, 11, 22, 1, 6]
>>> sorted(a)                    #这会从小到大排序, 不影响a原来的结构
[1, 6, 11, 22, 33, 66, 999]

>>> sorted(a, reverse = True)    #倒序
[999, 66, 33, 22, 11, 6, 1]

>>> b = sorted(a, reverse = True)  # sorted会返回一个新的列表
>>> b
[999, 66, 33, 22, 11, 6, 1]

>>> a.sort()                     #列表成员函数sort
>>> a
[1, 6, 11, 22, 33, 66, 999]		 #从小到大排序
>>> b = a.sort()                 #因为sort是直接影响原来结构,所以b不会有输出
>>> b

>>> c = ['addd','bcc','cb','da','zz']
# 在这里使用lamdba函数,就是匿名函数,取元素最后一位
# key, 以什么值来做排序的依据,按照元素的最后一个字母排序(ascii码从大到小)
>>> sorted(c, key=lambda x:x[-1])  
['da', 'cb', 'bcc', 'addd', 'zz']

>>> sorted(c, key=lambda x:x[-1], reverse=True)  # 倒序
['zz', 'addd', 'bcc', 'cb', 'da']

>>> sorted(c, key=len)
['cb', 'da', 'zz', 'bcc', 'addd']    #按照元素长度,从小大到,reverse=True 就可以倒序


#列表中的元素是元组
>>> d = [('b',22),('d',11),('a',33),('c',44)]
>>> sorted(d)
[('a', 33), ('b', 22), ('c', 44), ('d', 11)]   #默认按照元素的第一个值排序

>>> sorted(d, key=lambda x:x[1])
[('d', 11), ('b', 22), ('a', 33), ('c', 44)]   #指定按照元素的第二个值排序
```
