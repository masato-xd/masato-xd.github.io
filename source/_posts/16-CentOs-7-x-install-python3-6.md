---
title: CentOs 7.X下Python3与Python2共存
date: 2017-12-15 23:02:09
tags: 
- python 
- system
categories: 
- linux
---


{% note info %}
CentOs 7.X下Python3与Python2共存
{% post_link Win10共存Python2-Python3 Win10下共存可以点击这里 %}
{% endnote %}

<!-- more -->


**首先安装python3.6可能需要的依赖包**
`yum install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel`


**去官网下载最新的python安装包**
`wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz`
> 有一个注意点,如果直接使用`make intall` 会和系统自带的python安装到同一个目录/usr/bin/, 不好处理
> 这里我们首先创建一个自定义的目录,我这里是`/usr/local/python3`

**把python3.6安装到自定义目录**
--enable-shared参数就可以生成对应动态链接库
`./configure --prefix=/usr/local/python3 --enable-shared`
`make && make install`

**python3.6程序路径**
`python3.6程序的执行文件：/usr/local/python3/bin/python3.6`
`python3.6应用程序目录：/usr/local/python3/lib/python3.6`
`pip3的执行文件：/usr/local/python3/bin/pip3.6`
`pyenv3的执行文件：/usr/local/python3/bin/pyenv-3.6`


**系统自带的python就是一个软连接**
```bash
lrwxrwxrwx. 1 root root    7 7月  29 2016 /usr/bin/python -> python2
lrwxrwxrwx. 1 root root    9 7月  29 2016 /usr/bin/python2 -> python2.7
```

**修改默认软连接**
`cd /usr/bin/`
`mv python python.bak`
`ln -s /usr/local/python3/bin/python3.6 /usr/bin/python`
`ln -s /usr/local/python3/bin/pip3.6 /usr/bin/pip`

**删除软链接**
`rm -rf /usr/bin/pip`

**进入python**
` 输入Python 进入Python3 如果想用Python2的话 输入Python2 就行了`

**安装pip**
`/usr/bin/python -m pip install pillow`

**最后最关键的,修改yum文件,不然yum会无法使用**
`cd /usr/bin/
ls yum*`
```bash
yum  yum-builddep  yum-config-manager  yum-debug-dump  yum-debug-restore  yumdownloader  yum-groups-manager
```
修改这些程序头文件为`#!/usr/bin/python2`

**修改gnome-tweak-tool配置文件**
` vi /usr/bin/gnome-tweak-tool `
> `#!/usr/bin/python 改为 #!/usr/bin/python2`

**修改urlgrabber配置文件**
` vi /usr/libexec/urlgrabber-ext-down`
> `#!/usr/bin/python 改为 #!/usr/bin/python2`


**创建虚拟环境**
`python3 -m venv venv_name`
or
`pip install --user virtualenv (user实现用户隔离)`

example：
`python3 -m venv /opt/python_env`
`source /opt/python_env/bin/activate `
`deactivate`
or：
`virtualenv /opt/python_env`

----------
**资料：**
[1](http://blog.csdn.net/rubikta/article/details/76728434)
[2](http://blog.csdn.net/hobohero/article/details/54381475)