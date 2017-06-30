---
title: 'Mysql必知必会 <1> '
date: 2017-06-30 22:09:58
tags: mysql
categories: sql
---

### 创建用户
**命令:**
`CREATE USER 'username'@'host' IDENTIFIED BY 'password'; `

<!-- more -->

**说明:**
+ username - 你将创建的用户名
+ host - 指定该用户在哪个主机上可以登陆,如果是本地用户可用localhost, 如果想让该用户可以从任意远程主机登陆,可以使用通配符%
+ assword - 该用户的登陆密码,密码可以为空,如果为空则该用户可以不需要密码登陆服务器. 

**例子:**
> `CREATE USER 'cc'@'localhost' IDENTIFIED BY '123456'; `
`CREATE USER 'cc'@'192.168.1.101_' IDENDIFIED BY '123456'; `
`CREATE USER 'cc'@'%' IDENTIFIED BY '123456'; `
`CREATE USER cc IDENTIFIED BY '123456'; `
`CREATE USER 'cc'@'%' IDENTIFIED BY ''; `
`CREATE USER 'cc'@'%'; `

### 授权

创建的时候指定了主机名，授权的时候也需要使用用户名和主机名结合定义
**命令:**
`GRANT privileges ON databasename.tablename TO 'username'@'host'`
+ privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
+ databasename：数据库名
+ tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用\*表示，如\* . \*

**例子**
`GRANT ALL ON crashcourse.* TO 'cc'@'localhost'`

**查询权限**
`SHOW GRANTS FOR 'cc'@'localhost';`
**输出**:
```bash
+-----------------------------------------------------------------------+
| Grants for cc@localhost                                               |                                 					
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'cc'@'localhost' IDENTIFIED BY PASSWORD '*1111' |
| GRANT ALL PRIVILEGES ON `crashcourse`.* TO 'cc'@'localhost'           |
+-----------------------------------------------------------------------+
```

### 设置与更改用户密码 
**命令:**
`SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword')`

**如果是当前登陆用户用**
`SET PASSWORD = PASSWORD("newpassword"); `

**例子:**
`SET PASSWORD FOR 'cc'@'%' = PASSWORD("123456"); `


### 撤销用户权限 
**命令: **
`REVOKE privilege ON databasename.tablename FROM 'username'@'host'; `

**例子: **
`REVOKE SELECT ON *.* FROM 'cc'@'%'; `

> **注意: **
> 假如你在给用户`'cc'@'%'`授权的时候是这样的(或类似的):`GRANT SELECT ON test.user TO 'cc'@'%'`, 则在使用`REVOKE SELECT ON *.* FROM 'cc'@'%'`;命令并不能撤销该用户对test数据库中user表的SELECT 操作.相反,如果授权使用的是`GRANT SELECT ON *.* TO 'cc'@'%'`;则`REVOKE SELECT ON test.user FROM 'cc'@'%'`;命令也不能撤销该用户对test数据库中user表的Select 权限. 

### 删除用户
**命令:**
`DROP USER 'username'@'host';`