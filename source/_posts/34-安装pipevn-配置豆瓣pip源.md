---
title: '安装pipenv,配置豆瓣pip源'
date: 2019-04-14 18:51:31
tags: python
categories: python
---

{% note info %}

pipven是一种更高级的包管理工具, virtualenv、pyenv 和 pip 三者的功能于一身,python 版本需要3.x 以上

{%  endnote %}

<!-- more -->


> 安装pip及pipenv
```shell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
pip install pipenv
pipenv install --three
````


> 修改`Pipfile`文件，改为豆瓣软件源
```
[[source]]
url = "https://pypi.douban.com/simple/"
verify_ssl = true
name = "pypi"

[dev-packages]

[packages]
django = "==1.11"
ipython = "*"

[requires]
python_version = "3.6"
```

> 添加豆瓣源
```
## 创建编辑文件 
~/.pip/pip.conf

## 添加以下内容
[global]
timeout = 60
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```