---
title: 'Win10共存Python2,Python3'
date: 2017-07-03 00:47:50
tags: python
categories: python
---

有必要写一篇文章来分享一下设置p3和p2共存的文章，分享给大家
最后还有告诉大家怎么设置sub3的环境设置
[这是官网](https://www.python.org/)
<!-- more -->

> 我这边环境是先装的2，再装的3

选择安装的时候`Add Python to path`别忘了

{% asset_img toPath.png 添加到系统变量 %}

Python2一样的安装方法


----------


### 重点
网上很多资料是修改python.exe文件名，但是我们不用这们麻烦
**官方其实有解决方案**
在安装Python3(>=3.3)的时候，Python的安装包实际在**系统中安装了一个启动器py.exe**，默认`C:\Windows\`下面

当你使用:
下面运行的就是python2
> py -2 hello.py

下面运行的就是python3:
> py -3 hello.py

不想使用 - 2/3的时候，可以在**脚本的最上面**添加一行`#!python2` 或者 `#!python3`
> 定义编码`coding: utf-8`要写在第二行

```python
#!python2
#coding: utf-8
```
**这样就可以解决2和3的问题**


----------
### 搞定PIP的问题
使用 `pip -V`可以看到版本号(默认会按照最开始安装的pip来使用,所以pip 也能使用)
```powershell
pip -V
pip 8.1.2 from c:\python27\lib\site-packages\pip-8.1.2-py2.7.egg (python 2.7)
```

当使用`pip3 -V`会出现问题
```powershell
pip3 -V
Fatal error in launcher: Unable to create process using '"'
```

为了统一,我们就使用前面提到的py程序了
`py -2 -m pip -V`
```
pip 8.1.2 from C:\Python27\lib\site-packages\pip-8.1.2-py2.7.egg (python 2.7)
```

`py -3 -m pip -V`
```
pip 9.0.1 from C:\python36\lib\site-packages (python 3.6)
```

就可以通过这样来正常使用了


----------

### 接下来是编译器的问题
因为我使用的是Sublim Txt3,环境也需要设置一下

{% asset_img newBuildSystem.png 选择新建编译系统 %}

首先按照步骤`Tools > Build System > New Build System`
文件内容写入
```bash
# 注意是py的路径
	{
	    "cmd": ["C:/Windows/py", "-u", "$file"],
	    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
	    "selector": "source.python",
	}
```
保存到**User目录里面**,名字写个python3吧,其实通用的

以后编辑脚本的时候，在脚本最上面自定义要使用的编译器
{% asset_img python3.png python3 %}

{% asset_img python2.png python2 %}


----------


ok,全部搞定,有问题评论留言

----------
**补充一点pip警告的问题**
`DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a format=(legacy|columns) in your pip.conf under the [list] section) to disable this warning`

原则是不用管的,不会影响程序.但是遇到问题要解决它,我就是谷歌出来的,分享出来
在`C:\Users\你的账号`下建立pip文件夹，在pip下新建pip.ini：
> [list]
> format=columns

**或者:**
直接使用 pip freeze,  pip list 是将被弃用的

还是要解释一下大概意思:
> 它就是告诉你以后pip list的默认格式会采用columns，你可以在命令后面加上--format来指定什么展示格式

**解决方法:**
1. 加文件
2. 改用pip freeze。