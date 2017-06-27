---
title: 使用uWsgi启动Django
date: 2017-06-27 02:23:56
tags: django
categories: django
---


# 使用uWsgi启动Django

web应用部署，应该是对 web应用 + web应用服务器软件 + web服务器软件 的组合操作

**web应用**顾名思义就是你的web project，这可以是java web项目，**django项目**或者php项目；
**web应用服务器软件**就是运行web应用的地方，例如java用的tomcat服务器，**django用的uwsgi服务器**
正常情况下你只需要将**web项目**部署在**应用服务器软件**上就可以对外提供服务了

----------

本文主要介绍的是nginx + uwsgi 来为django应用提供服务
nginx 后续在补充,本文是直接使用uwsgi
之所以没有选择apache是因为公司的生产环境大部分是nginx做服务器软件的。
[这是官方中文文档](http://python.usyiyi.cn/django/howto/deployment/wsgi/uwsgi.html)


----------

<!-- more -->
## 使用--ini的方式
>好处是通过配置文件的方式启动,可以后台运行,灵活加载配置

当创建好了project以后，会在project下生成一个wsgi.py 的文件
![@我创建的一个project结构 ](/images/uwsgi.jpg)



----------

### 创建一个目录,负责存放ini的文件
我这里使用tailon,名字随意取得
然后创建存放pid(可以动态加载pid,不用kill 和重启 )的目录和日志的目录
下面是自定义的一个ini文件内容
>[uwsgi]
// project的路径
chdir=/root/xudeng/delay_web
// module：使用的WSGI 模块 —— 可以是startproject 创建的mysite.wsgi 模块。
module=delay_web.wsgi:application
// env 环境的路径
home=/root/xudeng/delay_web/dw_env
master=True
// 启动十个子进程
processes = 10
//端口号
http-socket=:9999
buffer-size=65535
vacuum=True
max-requests=5000
daemonize=/var/log/uwsgi/yourproject.log
//static-map= /static/=/home/qqdz/work/src/github.com/shwinpiocess/magina_project/magina/logs/static/
// 保存pid的路径
pidfile=/root/xudeng/delay_web/tailon/pid/test.pid


----------


### 启动和重新加载的命令
#### 启动命令
>uwsgi --ini tailon/test.ini

#### 重新加载命令
##### 使用简单选项 --reload
>uwsgi --reload tailon/pid/test.pid
##### 使用kill发送信号
>kill -HUP `cat /tmp/project-master.pid`
