---
title: python获取目录下的目录和文件
date: 2017-08-10 17:17:06
tags: python
categories: python
---
{% note info %}
<font color="red">**使用os.walk**</font>
<font color="red">**使用os.scandir()**</font>
{% endnote %}

<!-- more -->

**os.walk**
可以得到一个三元`tupple(dirpath, dirnames, filenames)`
第一个为起始路径，第二个为起始路径下的文件夹，第三个是起始路径下的文件。
`dirpath` 是一个string，代表目录的路径，
`dirnames` 是一个list，包含了dirpath下所有子目录的名字。
`filenames` 是一个list，包含了非目录文件的名字。
```python
import os

def get_full_path(path):
    for root, dirs, files in os.walk(path):
        for file_ in files:
            # 将路径和文件拼接起来
            print(os.path.join(root, file_))

if __name__ == '__main__':
    get_full_path('/path/to/some/folder')
```
----------
**os.scandir()**
Python 3.5版本中，新添加了`os.scandir()`方法，它是一个目录迭代方法。
在Python 3.5中，os.walk是使用os.scandir来实现的
根据Python 3.5宣称，它在POSIX系统中，执行速度提高了3到5倍
在Windows系统中，执行速度提高了7到20倍。
```python
import os

folders = []
files = []
 
for entry in os.scandir('/'):
    if entry.is_dir():
    	# 在这里直接使用entry.path显示路径
    	# 使用entry.name显示文件名
        folders.append(entry.path, entry.name)
    elif entry.is_file():
        files.append(entry.path)

print('Folders:')
print(folders)
```

----------
**例子:**

```python
#!python3
# coding: utf-8

import sys
import os

def main(path, ver):
    html_path = r'http://masato.com/'

    with os.scandir(path) as entrys:
        for entry in entrys:
            if entry.is_dir(): # 只需要目录
                print('\n' + "目录名称是: " + "[ " + entry.name + " ]")
                for root, dirs, files in os.walk(entry.path):
                    for file in files:
                    	# 获取文件的全路径
                        file_path = os.path.join(root, file)
                        # 将文件修改为"html_path"开头的连接
                        print(html_path + ver + '/' + ('/'.join(file_path.split('\\')[2:])))
    print('\n')

if __name__ == '__main__':
    try:
    	# 我的目录格式是这样的D:\测试\20170808R2_res47\，会自动切割最后的res47
        ver = sys.argv[1].split('_')[-1]
        main(sys.argv[1], ver)

    except FileNotFoundError as e:
        print('\n', e)

    except IndexError:
        print("\n" + "  Used: " + sys.argv[0] + " full_path" + "\n")

    except Exception as e:
        print('\n', e)


```
