---
title: ETCD alarm:NOSPACE 问题处理
date: 2018-01-26 00:31:13
tags:
- etcd
categories:
- db
---

{% note info %}

> etcd需要定期压缩以截断它的历史。etcd将检查它的db是否超过了空间配额限制，在这种情况下，它将抛出这个错误。
要从这个错误中恢复，首先要进行压缩，然后进行碎片整理，最后解除配额警报。[空间配额](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/maintenance.md#space-quota)文档有一个例子。

{% endnote %}

<!-- more -->

默认的空间配额使用完了以后就会报这个错误
**etcdserver: mvcc: database space exceeded**

#### 原因
因为ETCD每次的修改都会记录一次revision，方便回溯历史key记录，日积月累会越来越大
默认的后端配额是2G(最大可以设置到6G)

#### 确认警报信息
```bash
$ etcdctl --endpoints=$ENDPOINTS alarm list
memberID:13803658152347727308 alarm:NOSPACE 
```

#### 设置后端配额(根据需要配置)
设置一个很小的16MB的限额
`etcd --quota-backend-bytes=$((16*1024*1024))`


#### 获取当前的修订编号(reversion)，选取需要截断的历史
`rev=$(ETCDCTL_API=3 etcdctl --endpoints=:2379 endpoint status --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9]*')`

#### 进行压缩
`etcdctl --endpoints=$ENDPOINTS compact $rev`

#### 整理后端数据(文件系统)，释放空间
`etcdctl --endpoints=$ENDPOINTS defrag`

#### 解除警报
```bash
etcdctll --endpoints=$ENDPOINTS alarm disarm
```

#### 在启动的时候加上参数进行自动压缩回收
加上这个参数以后会自动进行`compact`，只需要定时执行`defrag`即可
默认是周期性的，1为小时(最新的版本支持分钟级别的`compact`)
`--auto-compaction-retention 1h`
`--auto-compaction-mode periodic|revision` 

#### 最后
官方建议最好通过周期性的压缩和碎片化整理来避免达到配额
而且没有自动触发`defrag`的机制，所以设置定时任务吧


----------
#### 资料

[ETCD原理分析](https://yuerblog.cc/2017/12/10/principle-about-etcd-v3/)
[mvcc error](https://github.com/coreos/etcd/issues/7986)
[配置文件](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/configuration.md)