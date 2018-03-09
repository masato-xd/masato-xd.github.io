---
title: 监控mongos连接数(python,shell)
date: 2018-03-10 00:05:21
tags: python
categories: python
---


{% note info %}

connections 当前连接和可用连接数，mongodb最大处理到2000个连接就不行了（要根据你的机器性能和业务来设定），所以设大了没意义。设个合理值的话，到达这个值mongodb就拒绝新的连接请求，避免被太多的连接拖垮。
分别使用Python和shell来实现,加入到zabbix监控模板中


{% endnote %}
<!-- more -->


**需要安装pymongo**
```
pip install pymongo
```

**进入命令行获取到得是这种格式**
```
mongos> db.serverStatus().connections
{
        "current" : 525,
        "available" : 523755,
        "totalCreated" : NumberLong(555796)
}
```

**python脚本的形式**
```python
#!/usr/bin/env python
# coding: utf-8

from pymongo import MongoClient

client = MongoClient('localhost', 30000)
db = client.db
conn = db.command("serverStatus")["connections"]['current']
client.close()

print conn
```

**使用Shell的方式，不需要安装pymongo**
```bash
#!/bin/sh

conn=`mongo 127.0.0.1:20000 --eval 'printjson(db.serverStatus().connections)' | grep -E '"current" : [0-9]*'| grep -o '[0-9]*'`

echo $conn
```

----------
**资料**
[How to use pytmongo get connections](https://stackoverflow.com/questions/34898723/how-to-get-output-of-db-serverstatus-connections-through-pymongo-in-python)

