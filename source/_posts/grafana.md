---
title: 无缝升级Grafana
date: 2017-06-30 00:10:37
tags: 随笔
categories: 小记
---


> No matter where your data is, or what kind of database it lives in, you can bring it together with Grafana. Beautifully.
> [下载地址](https://grafana.com/)

目前使用的Grafana是3.0.2的版本，现在版本到了4.3，加入了很多新特性，页面也更美观，让我们来升级吧
<!-- more -->

### 备份老的数据
单单在页面上将Dashbord　Export出来是没有用的，因为数据源是没法这样操作的，需要我们备份数据库文件、配置文件
> 配置文件:`/etc/grafana/grafana.init`
数据文件:`/var/lib/grafana/grafana.db`

### 下载
```powershell
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.3.2-1.x86_64.rpm 
yum localinstall grafana-4.3.2-1.x86_64.rpm 
```
通过这个命令就可直接升级本地Grafana

### 恢复
将备份的数据直接覆盖到目录，重启grafana-server服务。

### 提示
如果需要插件，可以去[插件中心](https://grafana.com/plugins)下载
升级以后如果某一个插件无法使用了，可以卸了再重新安装

**卸载插件**
`grafana-cli plugins remove <plugin-id>`

**升级某一个插件**
`grafana-cli plugins update <plugin-id>`

**安装最新的插件**
`grafana-cli plugins install <plugin-id>`

**升级所有插件**
`grafana-cli plugins update-all`