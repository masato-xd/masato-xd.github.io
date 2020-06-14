---
title: CMS服务备份与恢复
date: 2020-06-14 21:51:56
tags: 大数据
categories: 数据分析
---

{% note info %}

上篇文章中，详细说明了如何去部署CDH
实际生产中，CDH的重要服务都可以做HA（如HDFS、YARN）
但是CMS却只能单点，若CMS主机损坏或者服务损坏，不可能重新启动再接管集群，需要手动去恢复
今天模拟实际故障，演练CMS单点故障的处理方法和恢复。


{% endnote %}

<!-- more -->


## 1. 备份

这里建议使用完整备份，所以停止集群

生产环境肯定不能停止，所以只需要备份scm库即可

> 使用的是内嵌PG数据库
>
> scm用户密码在：`/etc/cloudera-scm-server/db.properties`
>
> cloudera-scm用户密码在： `/var/lib/cloudera-scm-server-db/data/generated_password.txt`

1. web页面，选择集群 > 停止

2. 选择cloudera-manager-server > 停止

3. CMS节点上，`service cloudera-scm-server stop`

4. `service cloudera-scm-agent stop`

5. `service cloudera-scm-server-db stop`

6. 打包压缩备份目录，`/var/lib/cloudera-scm-server`

7. 备份数据库

   ```bash
   pg_dump -h 127.0.0.1 -p 7432 -U scm > scm_server_db_backup.$(date +%Y%m%d)
   pg_dump -h 127.0.0.1 -U oozie_oozie_server -W -p 7432 oozie_oozie_server > oozie_oozie_server-backup-`date +%F`-CDH5.7.sql
   pg_dump -h 127.0.0.1 -U hive -W -p 7432 hive >  hive-backup-`date +%F`-CDH5.7.sql
   ```



## 2. 部署新的环境

在新的服务器上，只需要安装3个文件（需要提前下载放到一个目录中）

```bash
yum localinstall cloudera-manager-server-5.7.1-1.cm571.p0.8.el7.x86_64.rpm \
cloudera-manager-server-db-2-5.7.1-1.cm571.p0.8.el7.x86_64.rpm \
cloudera-manager-daemons-5.7.1-1.cm571.p0.8.el7.x86_64.rpm
```

安装`cloudera-manager-server-db-2`使用内嵌的数据库，也可以选择外置，根据线上环境定夺



## 3. 恢复

拷贝/var/lib/cloudera-scm-server/到新节点同级目录

```bash
cp cloudera-scm-server.tar.gz /var/lib/cloudera-scm-server/
```

赋权

```bash
chown cloudera-scm.cloudera-scm -R /var/lib/cloudera-scm-server/
```

启动DB

```
service cloudera-scm-server-db start
```

验证数据库

```bash
[root@cms-server ~]#  psql -h /run/cloudera-scm-server/ -p 7432  -U cloudera-scm  -d postgres
psql (9.2.24)
Type "help" for help.

postgres=# \l
                                                List of databases
        Name        |       Owner        | Encoding |  Collate   |   Ctype    |         Access privileges         
--------------------+--------------------+----------+------------+------------+-----------------------------------
 amon               | amon               | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 metastore          | hive               | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 nav                | nav                | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 navms              | navms              | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 oozie_oozie_server | oozie_oozie_server | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 postgres           | cloudera-scm       | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 rman               | rman               | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 scm                | scm                | UTF8     | en_US.UTF8 | en_US.UTF8 | 
 template0          | cloudera-scm       | UTF8     | en_US.UTF8 | en_US.UTF8 | =c/"cloudera-scm"                +
                    |                    |          |            |            | "cloudera-scm"=CTc/"cloudera-scm"
 template1          | cloudera-scm       | UTF8     | en_US.UTF8 | en_US.UTF8 | =c/"cloudera-scm"                +
                    |                    |          |            |            | "cloudera-scm"=CTc/"cloudera-scm"
(10 rows)

postgres=# 
```

创建数据库

> 模拟的环境上部署了hive和oozie服务

```
CREATE ROLE hive LOGIN PASSWORD 'hive';
CREATE DATABASE metastore OWNER hive ENCODING 'UTF8';
CREATE ROLE oozie_oozie_server LOGIN PASSWORD 'oozie_oozie_server';
CREATE DATABASE oozie_oozie_server OWNER oozie_oozie_server ENCODING 'UTF8';
```

恢复数据

```bash
psql -h /run/cloudera-scm-server/ -p 7432 -U cloudera-scm oozie_oozie_server < oozie_oozie_server-backup-2020-06-10-CDH5.7.sql
psql -h /run/cloudera-scm-server/ -p 7432 -U cloudera-scm metastore < hive-backup-2020-06-10-CDH5.7.sql
psql -h /run/cloudera-scm-server/ -p 7432 -U cloudera-scm scm <scm_server_db_backup.20200610
```

修改`pg_hba.conf`文件

> 以免访问数据库报错

```bash
echo "host all all 0.0.0.0/0 md5" >> /var/lib/cloudera-scm-server-db/data/pg_hba.conf 
# 修改后重新加载配置
sudo -u cloudera-scm pg_ctl reload -D /var/lib/cloudera-scm-server-db/data/
```

启动cm-server

```bash
service cloudera-scm-server start
```

修改所有节点agent配置并启动

```bash
vim /etc/cloudera-scm-agent/config.ini，#配置行：server_host=192.168.63.142

service cloudera-scm-agent start
```

启动后在管理web页面修改`hive`,`oozie`服务数据库配置