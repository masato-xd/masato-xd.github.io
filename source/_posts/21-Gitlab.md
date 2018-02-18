---
title: Gitlab的迁移、安装及恢复
date: 2018-02-18 12:27:41
tags:
categories:
---

{% note info %}

GitLab 是一个基于 git 的仓库管理程序，也是一个方便软件开发的强大完整应用。
在公司里搭建一套便于内部的代码管理，避免将代码暴露出去
本次项目需要进行迁移，记录本次操作，日后好回忆

{% endnote %}

<!-- more -->

**进行数据备份**
+ `gitlab-rake gitlab:backup:create`

执行这一条命令就会在默认备份目录生成按时间戳日期开头的备份tar包
```bash
# ls /var/opt/gitlab/backups/
1507543656_2017_10_09_gitlab_backup.tar 
```

**备份文件目录**
`/etc/gitlab/gitlab.rb` Gitlab的配置文件
`/var/opt/gitlab/nginx/conf` Nginx的配置文件
`/etc/postfix/main.cf` Postfix的配置文件
`/var/opt/gitlab/postgresql` Postgresql的文件(和用户数据有关)
> 我这里是将目录直接打包覆盖到新的

可以通过`/etc/gitlab/gitlab.rb`配置文件来修改默认存放备份文件的目录
`gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"`

**实现自动备份**
每天2点自动执行一次
+ `0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create`


----------

#### 搭建新的Gitlab环境

**安装依赖包和服务**
+ `yum install -y curl policycoreutils-python openssh-server cronie`
+ `yum install postfix`
+ `service postfix start`
+ `chkconfig postfix on`

**开启防火墙**
> 最好做限制，因为是针对公司使用的，不对外开放

+ `lokkit -s http -s ssh`  这里是暂且对外全开

**安装下载Gitlab包**
> 官网指导会安装EE版本(企业版)，公司自用就使用CE(社区版免费的)

+ `curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash`
+ `yum install gitlab-ce`

> 这里我安装的时候碰到一个问题，下载巨慢，好在谷歌搜到的解决办法，自己下载包，本地安装，感谢这位同学
1. [gitlab yum/apt安装很慢的解决办法](http://www.doocr.com/articles/58b15886f8694d05da412c89)
2. [GitlabRPM官网](https://packages.gitlab.com/gitlab/gitlab-ce)

> 下载好了以后，使用`yum localinstall package`即可

**启动**
+ `gitlab-ctl reconfigure`
+ `gitlab-ctl status`
+ `gitlab-ctl stop`
+ `gitlab-ctl start`


----------
#### 恢复数据
> **重要提醒：**Gitlab版本必须一致
> 查看方法：
> 1.`head -1 /opt/gitlab/version-manifest.txt`
> 2.`cat /opt/gitlab/embedded/service/gitlab-rails/VERSION`

**导入之前备份的文件**
如果没有修改过默认配置，将备份的文件放到默认目录中
`/var/opt/gitlab/backups/`

**关闭服务**
停止相关数据连接服务
`gitlab-ctl stop unicorn`
`gitlab-ctl stop sidekiq`

停止相关服务
`gitlab-ctl stop`
`gitlab-ctl stop nginx`

**恢复数据**
`gitlab-rake gitlab:backup:restore BACKUP=1507543656_2017_10_09`

**启动即可**
`gitlab-ctl start`


----------

相关资料
[Gitlab备份恢复迁移](http://www.xuliangwei.com/xubusi/803.html)
[Yum命令使用说明](http://blog.csdn.net/samxx8/article/details/47448479)
[Gitlab官网](https://about.gitlab.com/installation/#centos-6)