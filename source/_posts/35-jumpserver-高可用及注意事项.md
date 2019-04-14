---
title: jumpserver 高可用及注意事项
date: 2019-04-14 19:08:41
tags: system
categories: linux
---

{% note info %}

堡垒机,为了保证服务器安全，加个堡垒机，所有ssh连接都通过堡垒机来完成，
堡垒机也需要有身份认证，授权，访问控制，审计等功能。
Jumpserver 是一款由python编写开源的跳板机（堡垒机）系统，实现了跳板机应有的功能。
基于ssh协议来管理，客户端无需安装agent
感谢伟大的开源软件贡献者

{% endnote %}

<!-- more -->


### 基本结构
#### 架构图

{% asset_img 1.png 架构图 %}

#### 服务器信息
搭建两套jumpserver，两套coco，环境一模一样

| 主机 |ip  |角色|
| --- | --- | --- |
| ly-txds-3p-3j-jump1 | 42.51.7.61  |master|
| tz-jump1 | 122.226.181.102  |slave|


### 安装
#### 安装Jumpserver
官方文档非常详尽，不再赘述
[官方链接](http://docs.jumpserver.org/zh/docs/setup_by_ubuntu.html)
[官方Github-Issue](https://github.com/jumpserver/jumpserver/issues?page=2&q=is%3Aissue+is%3Aopen)

#### 配置Mysql主主
安装`Jumpserver`中有步骤安装`Mysql`，不再赘述，主要是`Mysql`配置
```
## 编辑文件，master与slave都修改bind-adress参数，监听本地所有IP
vim /etc/mysql/mysql.conf.d/mysqld.cnf 
bind-address=0.0.0.0

## 在master角色，文件最后添加以下参数
vim /etc/mysql/mysql.conf.d/mysqld.cnf 
[mysqld]
server-id=31 #服务器ID，不能重复
## 业务有要求，ID必须连续，那就不能设置这两个参数了
auto-increment-offset=1 # 自增因子
auto-increment-increment=2 # 自增数量，奇数ID

log-bin=mysql-bin # 开启master功能，做master都需要开启此参数
binlog_format=mixed # binlog的格式
relay-log=relay-bin
log-slave-updates=1 #将复制事件写入binlog,一台服务器既做主库又做从库此选项必须要开启
replicate-ignore-db = mysql,information_schema,performance_schema # 忽略不同步的库，一般为了保证主主同步不冲突，会不同步mysql数据库。

## 在slave角色，文件最后添加以下参数 
vim /etc/mysql/mysql.conf.d/mysqld.cnf 
[mysqld]
server-id=32 #服务器ID，不能重复
## 业务有要求，ID必须连续，那就不能设置这两个参数了
auto-increment-offset=2 # 自增因子
auto-increment-increment=2 # 自增数量，偶数ID

log-bin=mysql-bin # 开启master功能，做master都需要开启此参数
binlog_format=mixed # binlog的格式
relay-log=relay-bin
log-slave-updates=1 #将复制事件写入binlog,一台服务器既做主库又做从库此选项必须要开启
replicate-ignore-db = mysql,information_schema,performance_schema # 忽略不同步的库，一般为了保证主主同步不冲突，会不同步mysql数据库。
```
#### 创建同步账号
```mysql
# master上执行
mysql > grant replication slave,replication client on *.* to 'copy'@slave_ip identified by '123456';
mysql > flush privileges;

# slave上执行
mysql > grant replication slave,replication client on *.* to 'copy'@master_ip identified by '123456';
mysql > flush privileges;
```

#### 获取master信息
```
## 因为互为主备，所以两边都需要获取bin-log-file，position信息
# master上执行
mysql > show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |   1111 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

# slave上执行
mysql > show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |   2222|              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

#### 配置主库信息
```mysql
# master配置slave为主库
mysql > CHANGE MASTER TO 
MASTER_HOST='slave_ip' ,
MASTER_USER='copy',
MASTER_PASSWORD='123456' ,
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=2222;

# slave配置master为主库
mysql > CHANGE MASTER TO 
MASTER_HOST='master_ip',
MASTER_USER='copy',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=1111;,
```

#### 启动从库
```mysql
# 执行命令启动从库
start slave;
show slave status\G

# 确认以下两个参数为YES代表同步正常
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

#### 启动Jumpserver
> **启动JMS服务，需要先启动jms，再启动coco**
> **停止JMS服务，需要先停止coco，再停止jms**
```shell
 # 启动mysql
 systemctl start mysql
 
# 启动jms
cd /opt/jumpserver/
./jms start -d

# 启动coco
cd /opt/coco/
 ./cocod start -d

 # 启动nginx
 systemctl start nginx
```


### 排错思路
通过以下两点确定服务是否异常
1.  web页面发出的错误信息
2.  连接coco跳转到服务器发出的错误信息

无法连接到coco排错思路
1.  用户本地的私钥key错误
2.  coco服务没有启动

> *日志提示登录频繁*
> 是因为用户被锁定了，在web页面解除用户限制

#### 主要文件
```
# jumpserver配置文件
/opt/jumpserver/config.yml
/opt/coco/config.yml 

# jumpserver主要log文件
/opt/coco/logs/jump1.log
/opt/jumpserver/logs/*.log

# mysql配置文件
/etc/mysql/mysql.conf.d/mysqld.cnf 

# mysql日志文件
/var/log/mysql/error.log
/var/log/mysql.log
```

#### 需要做的操作

 `备机Jumpserver`在正式使用时，需将 `系统设置 -- 基本设置 -- 当前站点URL`的IP地址更正为备机IP，这样通过备机注册用户的时候收到的URI为备机IP路径
 
 启动`备机Jumpserver`的时候，因为是主主模式，所以不用修改数据库配置
 
 
#### 从0.3.x版本迁移，需要注意的

1. 注意管理用户的公钥与私钥
   这里踩了一个大坑，旧JMS管理用户的key
   真实路径是`/usr/local/src/jumpserver/keys/default/admin_user.pem`，并不在`/home/jump_super/.ssh`下
   
2. 新搭建的JMS服务器`系统中`需要有旧JMS管理用户账户，将新JMS的公钥和私钥替换为旧的，这样不影响client上的用户
3. client服务器，防火墙要允许新JMS服务器的IP访问
4. 注意client服务器的ssh端口


#### 参考资料
[MySql双主架构](http://blog.51cto.com/ygqygq2/1870864)
[主主同步参数](https://blog.csdn.net/solaraceboy/article/details/79903153)
[jms备机操作概要](https://wuyanteng.github.io/2018/11/28/Jumpserver1-4-3%E5%A4%87%E6%9C%BA-%E9%83%A8%E7%BD%B2%E6%A6%82%E8%A6%81/)


### 正确删除从库的步骤
```

正确关闭从库
正确关闭slave步骤
1. 执行STOP SLAVE语句
2. 使用SHOW STATUS检查slave_open_temp_tables变量的值
3. 如果值为0，使用mysqladmin shutdown命令关闭从服务器
4. 如果值不为0，用START SLAVE重启从服务器线程
slave_open_temp_tables值显示，当前slave创建了多少临时表，注意由client显示创建的
即便是这样，在使用临时表的场景下，如果服务器宕机，将遇到不可预知的问题。
所以比较保险的做法是，创建实体表，虽然会由于分配的文件刷新到磁盘。

mysql> show status like '%slave%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| Com_show_slave_hosts   | 0     |
| Com_show_slave_status  | 0     |
| Com_slave_start        | 0     |
| Com_slave_stop         | 0     |
| Slave_open_temp_tables | 0     |
+------------------------+-------+
5 rows in set (0.00 sec)
```