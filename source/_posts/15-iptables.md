---
title: 使用iptables查看数据流量
date: 2017-09-07 23:31:30
tags: system
categories: linux
---

{% note info %}
要监控系统中某一个IP,某一个端口的流量,不需要第三方的工具
使用iptables就可以实现
{% endnote %}

<!-- more -->

**命令:**
```bash
iptables -L -v -n -x INPUT 
```
> 说明: 这条命令会显示当前iptables所有规则的情况
> `-L:`列出规则链中所有的规则
> `-v:`显示详细信息,包含通过该规则的数据包数量，总字节数及相应的网络接口
> `-n:`不进行域名解析,显示IP
> `-x:`不进行单位转换(默认是bytes)
> `INPUT`: 指定显示INPUT关卡

**结果:**
```
[root@test ~]# iptables -vnL  INPUT -t filter
Chain INPUT (policy DROP 725 packets, 48678 bytes)
 pkts bytes target     prot opt in     out     source               destination   
     0     0            udp  --  *      *       0.0.0.0/0           9.8.4.2        udp dpt:9527       
38740 6613K            tcp  --  *      *       0.0.0.0/0            9.8.4.2        tcp dpt:9527 
 5310  379K ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
1692K  573M ACCEPT     all  --  eth1   *       0.0.0.0/0            0.0.0.0/0           
  146 10894 ACCEPT     all  --  *      *       9.2.54.226         0.0.0.0/0           
    0     0 ACCEPT     all  --  *      *       9.2.54.227         0.0.0.0/0           
   0     0 ACCEPT     tcp  --  *      *       22.7.3.16        0.0.0.0/0           tcp dpt:22 
    0     0 ACCEPT     tcp  --  *      *       11.94.4.5        0.0.0.0/0           tcp dpt:10050 
 263M  105G ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
14731  645K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:80 
  53M 3267M ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           tcp dpt:9527 
 141K 8826K DROP       icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      *       22.7.3.14        0.0.0.0/0                     
```
>  解析: 这是测试的数据,可以看到我们指定显示filter表中的INPUT链的结果
> `policy DROP:`代表的是当前链默认的策略,在这里进入的所有流量都DROP掉(等于是开启白名单)
> `packets:`当前链默认策略匹配到的包的数量
> `bytes:`当前链默认策略匹配到的所有包的大小总和
>  我们可以把packets和bytes当做"计数器", 计数器记录了默认策略匹配到的报文数量与总大小只有在使用`-v`选项时,才会显示出来


----------

上面的命令是查看规则(默认是filter表),下面我们来添加自己需要的规则
```
iptables -I INPUT -p udp -d 9.8.4.2 --dport 9527
```
> 说明:插入一条规则(没有指定-t 表名的情况下, 默认是filter表)
> `-I:`表示在INPUT链中插入一条规则(首部插入规则)
> `-p:`指定报文的协议
> `-d:`报文请求的目的IP地址
> `--dport:`报文请求的目的端口

添加完了以后我们就可以使用`iptables -vnL  INPUT -t filter`查看到指定规则的数据情况