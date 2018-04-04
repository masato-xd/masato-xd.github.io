---
title: 记一次服务器nf_conntrack:table_full
date: 2018-04-04 17:55:25
tags: system
categories: linux
---

{% note info %}

`nf_conntrack`模块用于跟踪连接的状态，供其他模块使用(调用信息)
最常见的使用场景是` iptables` 的 `nat` 和 `state` 模块
`nf_conntrack`用1个`HASH表`记录已建立的连接，包括其他机器到本机、本机到其他机器、本机到本机
如果连接进来比释放的快，把哈希表塞满了，此时`netfilter`变成了一个黑洞，导致丢弃任何数据包。

{% endnote %}

<!-- more -->

#### 现象:

曲线异常,人数持续下降,与此同时服务器时通时不通

#### 问题:

查看日志发现大量异常:

```bash
#cat /var/log/message
kernel: nf_conntrack: table full, dropping packet
#grep conntrack /var/log/messages
nf_conntrack version 0.5.0 (16384 buckets, 65536 max)
```

#### 原因:

服务器访问量大，内核`netfilter`模块`conntrack`相关参数配置不合理，导致新连接被`drop`掉。
这里看到上面最大值是`65536`,  线上3台服务器差不多最大能支持190008个连接.
当时情况的确有19w+的连接,所以出现问题.

#### 分析:

`nf_conntrack`模块用于跟踪连接的状态，供其他模块使用(调用信息)
最常见的使用场景是`iptables` 的 `nat` 和 `state` 模块
`nf_conntrack`用1个`HASH表`记录已建立的连接，包括其他机器到本机、本机到其他机器、本机到本机
如果连接进来比释放的快，把哈希表塞满了，此时`netfilter`变成了一个黑洞，导致丢弃任何数据包。 

#### 命令:

`netfilter` 相关的内核参数：
```bash
# 实时连接跟踪数(这个可以做到监控中)
sysctl net.netfilter.nf_conntrack_count
# 查看最大连接数(故障的时候是65536,net.netfilter.nf_conntrack_buckets*4)
sysctl  net.netfilter.nf_conntrack_max
# HASH表的大小(net.netfilter.nf_conntrack_max/4)
# 根据实际看到,hash表的大小并不会影响到连接
# 重点还是max的值会影响连接
sysctl net.netfilter.nf_conntrack_buckets
```

`max`的计算公式是:
```bash
# CONNTRACK_MAX = RAMSIZE (in bytes) / 16384 / (ARCH / 32)
# ARCH为你机器CPU的架构，64或32）
net.netfilter.nf_conntrack_max = 48(G) * 1024*1024*1024/16384/2 
```

`hashsize`的计算公示是:
```bash
# 哈希表大小（只读）（64位系统、8G内存默认 65536，16G翻倍，如此类推）
net.netfilter.nf_conntrack_max
```

#### 优化:

调大`nf_conntrack_buckets`和`nf_conntrack_max`,超时时间这里没有考虑
```bash
# 机器是128G内存的 
# vim /etc/sysctl.d/99-sysctl.conf 这个文件是 /etc/sysctl.conf 软连接
net.netfilter.nf_conntrack_max = 4194304
net.nf_conntrack_max = 4194304

# net.netfilter.nf_conntrack_buckets 不能直接改(报错)
echo 1048576 > /sys/module/nf_conntrack/parameters/hashsize

# 最后执行
sysctl -p
```

#### 相关资料:

[Conntrack tuning](https://wiki.khnet.info/index.php/Conntrack_tuning)
[nf_conntrack_max优化](https://testerhome.com/topics/7509)

