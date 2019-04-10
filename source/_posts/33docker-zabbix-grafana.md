---
title: docker搭建zabbix+grafana
date: 2019-04-10 11:35:40
tags: docker
categories: 容器
---

{% note info %}

`docker`基于容器化，沙箱机制，可使你用较少的命令和脚本快速部署应用。一次构建，多处移植使用。再配合shell等脚本语言，可实现脚本化一键部署。

{% endnote %}

<!-- more -->


#### 文献资料
[通过容器启动zabbix](
https://www.zabbix.com/documentation/4.2/manual/installation/containers)
[通过容器启动grafana](
https://grafana.com/docs/installation/docker/)

#### zabbix-server
> 根据官方文档,可以使用多种形式创建
> `server`与`db`与`web`分开，通过`link`方式连接，在这里使用最易于实现的方式

```shell
docker run --name zabbix-appliance -t \
-p 10051:10051 \
-p 80:80 \
-d zabbix/zabbix-appliance:latest
```

#### zabbix-agent

```shell
docker run --name zabbix-server-agent \
-e ZBX_HOSTNAME="192.168.101.140" \
-e ZBX_SERVER_HOST="192.168.101.140,172.17.0.1" \
-p 10050:10050 \
-d zabbix/zabbix-agent:alpine-4.2-latest
```
#### grafana

使用兼顾数据持久化的方式启动
```bash
#!/bin/bash
basedir=$(cd `dirname $0`;pwd)

mkdir -p data # creates a folder for your data
ID=$(id -u) # saves your user id in the ID variable

docker stop grafana
docker rm grafana
docker run \
       -d --name grafana  -p 3000:3000 \
       -e "GF_SECURITY_ADMIN_PASSWORD=lingdasa" \
       --user $ID --volume "$PWD/data:/var/lib/grafana" \
       grafana/grafana grafana
```

#### 问题

> zabbix前端时区的问题，两个方式，推荐使用方法2

1, 重启容器以后，时区会还原，需要修改配置文件

```bash
vi /etc/php5/conf.d/99-zabbix.ini
date.timezone=Asia/Shanghai
```
2, 通过指定`PHP_TZ`变量启动容器 `-e PHP_TZ=Asia/Shanghai`

> agent日志报，拒绝172.17.0.1主机连接的错误
> 原因是因为容器生成IP不是宿主机的ip,需要指定server_ip为容器的IP

```
-e ZBX_SERVER_HOST="192.168.101.140,172.17.0.1"
```

