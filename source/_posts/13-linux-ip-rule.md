---
title: Linux下的策略路由设置
date: 2017-08-15 21:22:16
tags: system
categories: linux
---


{% note info %}
**策略路由**
策略性是指对于IP包的路由是以系统管理员根据需要定下的一些策略为主要依据进行路由的。
例如我们可以有这样的策略："当服务器有两块网卡的时候,A网卡来的数据从A出去,B网卡来的数据从B出去,做到源进源出"
{% endnote %}

<!-- more -->


```bash
#!/bin/bash 

sed -i  '/252 bgp/d' /etc/iproute2/rt_tables 
sed -i  '/251 yidong/d' /etc/iproute2/rt_tables 

IF_BGP="bond0.1503"
IF_YIDONG="bond0.1501"
GW_BGP=103.19.255.193
GW_YIDONG=117.18.36.129
IP_BGP=103.19.255.236
IP_YIDONG=117.18.36.216

# 打开linux的路由功能
echo '1' > /proc/sys/net/ipv4/ip_forward 

# 在rt_tables中添加自定义路由表, 表序号和表名字
echo '252 bgp' >> /etc/iproute2/rt_tables 
echo '251 yidong' >> /etc/iproute2/rt_tables 

# 刷新bgp\yidong路由表
ip route flush table bgp
ip route flush table yidong

# 添加一个路由规则到 bgp 表,这条规则是 bgp 这个路由表中的数据包 默认通过网卡 bond0.1501 经过网关 103.19.255.193 出去
ip route add default via $GW_BGP dev $IF_BGP table bgp

# 添加一个路由规则到 yidong 表,这条规则是 yidong 这个路由表中的数据包 默认通过网卡 bond0.1503 经过网关 117.18.36.129 出去
ip route add default via $GW_YIDONG dev $IF_YIDONG table yidong

#来自 103.19.255.236 的数据包，使用 bgp 路由表的路由规则
ip rule add from $IP_BGP table bgp

#来自 117.18.36.216 的数据包，使用 yidong 路由表的路由规则
ip rule add from $IP_YIDONG table yidong

# 打了标记1的走BGP路由表,打了标记2的走YIDONG路由表
# 此句型配合iptables -t mangle应用.如先对数据包作标记:
ip rule add fwmark 1 table bgp 
ip rule add fwmark 2 table yidong 
```
