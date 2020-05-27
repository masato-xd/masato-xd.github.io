---
title: 解决linux删除文件后空间未释放
date: 2020-02-10 13:03:43
tags: system
categories: linux
---

{% note info %}

linux有时会遇到一种情况，某个文件删除了,但是磁盘空间并没有释放
这是因为仍然有进程在使用此文件，其文件句柄没有被释放
两种方式来解决此问题。

{% endnote %}

<!-- more -->

**重启服务，释放文件句柄**

1.使用`lsof`找到进程

```bash
root# lsof | fgrep "deleted"
```

2.根据上面得到的`pid`，进行进程重启或直接`kill`此进程，此时空间会被释放



**不重启服务**

1.使用`lsof`找到进程

2.找到文件描述符（pid）所在地

```bash
root#ls -l /proc/${pid}/fd/ | fgrep "deleted"
```

3.截断文件描述符

```bash
# 清空fd
> /proc/${pid}/fd/${id}
# 或直接调用truncate命令
truncate -s 0 /proc/${pid}/fd/$
```

