---
title: 记一次su - <username>需要密码
date: 2018-03-09 23:51:21
tags: system
categories: linux
---

{% note info %}

#### 故障描述:

  服务器在一次重启后，发现`root `使用 `su - (本机任意用户)`需要密码

{% endnote %}

<!-- more -->

#### 涉及排查的日志:

```bash
cat /etc/pam.d/su
cat /etc/pam.d/system-auth
cat /etc/pam.d/password-auth
cat /home/qqdz/.bashrc
cat /home/qqdz/.bash_profile
less /var/log/messages

# 下面是发现问题的重要日志
less /var/log/audit/audit.log
tailf /var/log/secure
cat /etc/profile
```

#### 主要的报错信息:

```bash
# tailf /var/log/audit/audit.log
type=USER_AUTH msg=audit(1520490151.355:3436): pid=66888 uid=0 auid=0 ses=106 msg='op=PAM:authentication grantors=pam_rootok acct="qqdz" exe="/usr/bin/su" hostname=? addr=? terminal=pts/1 res=success'
type=USER_ACCT msg=audit(1520490151.355:3437): pid=66888 uid=0 auid=0 ses=106 msg='op=PAM:accounting grantors=pam_succeed_if acct="qqdz" exe="/usr/bin/su" hostname=? addr=? terminal=pts/1 res=success'
type=CRED_ACQ msg=audit(1520490151.355:3438): pid=66888 uid=0 auid=0 ses=106 msg='op=PAM:setcred grantors=pam_rootok acct="qqdz" exe="/usr/bin/su" hostname=? addr=? terminal=pts/1 res=success'
type=USER_START msg=audit(1520490151.357:3439): pid=66888 uid=0 auid=0 ses=106 msg='op=PAM:session_open grantors=pam_keyinit,pam_keyinit,pam_limits,pam_systemd,pam_unix,pam_xauth acct="qqdz" exe="/usr/bin/su" hostname=? addr=? terminal=pts/1 res=success'

type=USER_AUTH msg=audit(1520490153.285:3440): pid=66923 uid=1000 auid=0 ses=106 msg='op=PAM:authentication grantors=? acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/1 res=failed'

```

```bash
# less /var/log/secure
Mar  8 13:11:57 qqdzz-Hadoop7 su: pam_unix(su-l:session): session opened for user ams by (uid=0)
Mar  8 13:11:57 qqdzz-Hadoop7 su: pam_unix(su-l:auth): auth could not identify password for [root]
Mar  8 13:11:57 qqdzz-Hadoop7 su: pam_succeed_if(su-l:auth): requirement "uid >= 1000" not met by user "root"

```

#### 分析现象:

发现疑点是在`audit.log`记录日志中，`su - qqdz`其实已经切换到了`qqdz`用户,加载了环境变量
但是又立即执行了 `su - root`，导致需要输入密码。
这个时候其实是不影响进入`qqdz`用户的，但是远程用户会很麻烦（本来是无秘钥登录，现在需要密码）

#### 找到原因:

与项目确认是有改过`/etc/profile`文件，导致切换用户会立即执行

项目在`/etc/profile(全局环境变量文件，用户登录以后立即加载)`中添加上了一个服务启动，必须要root用户执行

文件中有 ===>  service ambari-agent start 

#### 解决问题:

将`service ambari-agent start` 放入`rc.local`中,开机启动

#### 进一步分析:

查看 ambari-agent 服务中的配置：

```bash
# cat /etc/ambari-agent/conf/ambari-agent.ini    
[server]
hostname=hadoop1-ambari-v2
url_port=8440
secured_url_port=8441

[agent]
prefix=/var/lib/ambari-agent/data
;loglevel=(DEBUG/INFO)
loglevel=INFO
data_cleanup_interval=86400
data_cleanup_max_age=2592000
data_cleanup_max_size_MB = 100
ping_port=8670
cache_dir=/var/lib/ambari-agent/cache
tolerate_download_failures=true
run_as_user=root   # 本次故障受到这个选项影响
parallel_execution=0
alert_grace_period=5
alert_kinit_timeout=14400000
system_resource_overrides=/etc/resource_overrides

[security]
keysdir=/var/lib/ambari-agent/keys
server_crt=ca.crt
passphrase_env_var_name=AMBARI_PASSPHRASE

[services]
pidLookupPath=/var/run/

[heartbeat]
state_interval=6
dirs=/etc/hadoop,/etc/hadoop/conf,/etc/hbase,/etc/hcatalog,/etc/hive,/etc/oozie,
  /etc/sqoop,/etc/ganglia,
  /var/run/hadoop,/var/run/zookeeper,/var/run/hbase,/var/run/templeton,/var/run/oozie,
  /var/log/hadoop,/var/log/zookeeper,/var/log/hbase,/var/run/templeton,/var/log/hive
; 0 - unlimited
log_lines_count=300

[logging]
syslog_enabled=0

```