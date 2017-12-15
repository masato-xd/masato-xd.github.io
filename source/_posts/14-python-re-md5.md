---
title: python正则取值以及生成字串的MD5
date: 2017-08-25 11:04:19
tags: python
categories: python
---

{% note info %}
使用`python`内置`hashlib`库对字串做MD5加密,使用`re`库做正则取值
{% endnote %}

<!-- more -->


```python
# 导入hashlib库
import hashlib
# 生成MD5对象
m = hashlib.md5()
# 传入需要加密的字串
m.update("str4MD5Encode")
# 获得加密后的结果
encodeStr = m.hexdigest()
# 打印输出
print encodeStr
f8fd73cf519e6f11513d505b9dd33541
```

**为了复用，写一个简单的函数**
```python
def md5Encode(str):
    m = hashlib.md5()
    m.update(str)
    return m.hexdigest()
```


{% note danger %}
**注意**
这里写成一个函数有两个好处
- 方便重复使用
- 运行`m.update(str)`以后，如果再次运行`m.update(new_str)`，结果将会是`m.update(str+new_str)`

所以我们使用写成函数，传入准确的字串，获取MD5
{% endnote %}

----------


**示例文本:**
```bash
# testFile
100000637 {"device_id":"864836023415302","mac":"44:91:57:0c:71:ce"}
100000638 {"mac":"f0:21:6c:ba:d3:98","device_id":"860440031505974"}
100000662 {"mac":"40:c6:2a:22:0e","device_id":"868904020276766"}
100000673 {"mac":"94:92:bc:da:ca:93","device_id":"869735020030046"}
100000684 {"mac":"fc:1a:11:ab:6d:f4","device_id":"860736036152609"}
100000731 {"device_id":"a0:cd:c6:49:48:5a","mac":"a0:86:2d:49:48:5a"}
100000738 {"device_id":"864895024394247","mac":"00:E0:de:24:C9:00"}
100000752 {"mac":"28:12321:1c:72:62","device_id":"868970028781214"}
100000754 {"device_id":"866641029299231","mac":"18:59:231:b5:77:dc"}
100000788 {"device_id":"866401021328946","mac":"7c:1dg:59:1f:58"}
```

**脚本:**
```python
# coding: utf-8
import hashlib
import re

def md5Encode(str):
    m = hashlib.md5()
    m.update(str)
    return m.hexdigest()

reDevice = re.compile(r'"(device_id)":"(.*?)"')

with open('testFile') as f1, open('id_md5', 'w') as f2:
    for line in f1:
        # 通过正则pattern对象的search()方法, 对分组数据进行MD5加密
        device_id = reDevice.search(line.strip()).group(2)
        id_md5 = md5Encode(device_id)
        f2.write(device_id + ":" + id_md5 + '\n')
```

**结果:**
```bash
# cat id_md5
864836023415302:a5ade0da66ad15a80e91438f73289ee7
860440031505974:5f58bcb19a30875f27bbfa61b5edf712
868904020276766:d139843333d646ce4fcdf7841244cfab
869735020030046:da542e0f71b3c4ac47436d59dd65f4a5
860736036152609:f3970ffd140a252c145755f5397debdd
a0:86:c6:49:48:5a:71fd88a02db8529ae62844fefcab99ef
864895024394247:94f29934a622f4154c1456fcf4fdee81
868970028781214:39521a4b53f33c7d56ce6bf9e6b1990f
866641029299231:b4b6fd43f950d74053e18a56413d85b2
866401021328946:75b2c413c3f8e2fc8338f4f3bf8c00ff
```