---
title: MongoDB集群配置
date: 2018-01-26 00:23:34
tags:
- mongodb
categories:
- db
---

{% note info %}

{% asset_img mongo.png %}

从图中可以看到有四个组件：`mongos、config server、shard、replica set`。

{% endnote %}

<!-- more -->

**mongos**，数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求请求转发到对应的shard服务器上。在生产环境通常有多mongos作为请求的入口，防止其中一个挂掉所有的mongodb请求都没有办法操作。

**config server**，顾名思义为配置服务器，存储所有数据库元信息（路由、分片）的配置。mongos本身没有物理存储分片服务器和数据路由信息，只是缓存在内存里，配置服务器则实际存储这些数据。mongos第一次启动或者关掉重启就会从 config server 加载配置信息，以后如果配置服务器信息变化会通知到所有的 mongos 更新自己的状态，这样 mongos 就能继续准确路由。在生产环境通常有多个 config server 配置服务器，因为它存储了分片路由的元数据，防止数据丢失！

**shard，分片（sharding）**是指将数据库拆分，将其分散在不同的机器上的过程。将数据分散到不同的机器上，不需要功能强大的服务器就可以存储更多的数据和处理更大的负载。基本思想就是将集合切成小块，这些块分散到若干片里，每个片只负责总数据的一部分，最后通过一个均衡器来对各个分片进行均衡（数据迁移）。

**replica set**，中文翻译副本集，其实就是shard的备份，防止shard挂掉之后数据丢失。复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。

**仲裁者（Arbiter）**，是复制集中的一个MongoDB实例，它并不保存数据。仲裁节点使用最小的资源并且不要求硬件设备，不能将Arbiter部署在同一个数据集节点中，可以部署在其他应用服务器或者监视服务器中，也可部署在单独的虚拟机中。为了确保复制集中有奇数的投票成员（包括primary），需要添加仲裁节点做为投票，否则primary不能运行时不会自动切换primary。

简单了解之后，我们可以这样总结一下，应用请求mongos来操作mongodb的增删改查，配置服务器存储数据库元信息，并且和mongos做同步，数据最终存入在shard（分片）上，为了防止数据丢失同步在副本集中存储了一份，仲裁在数据存储到分片的时候决定存储到哪个节点。

----------


**服务结构规划**

| mongo1	| mongo2 	|   mongo3	 |
| :--------:| :--------:| :-------:  |
| mongos  	| mongos	|  mongos	 |
| config server|config server| config server |
| shard server1 主节点|shard server1 副节点|shard server1 仲裁|
| shard server2 仲裁|shard server2 主节点|shard server2 副节点|
| shard server3 副节点|shard server3 仲裁|shard server3 主节点|


**mongo-ip**
```
mongo1	10.20.49.142
mongo2	10.20.49.143
mongo3	10.20.49.144
```


**端口分配**
```
mongos：30000
config：31000
shard1：32001
shard2：32002
shard3：32003
```

**创建目录**
`mkdir -p config/data config/log mongos/data mongos/log shard1/data shard1/log shard2/data shard2/log shard3/data shard3/log`

**安装mongodb**
安装mongodb很简单,官网下载包以后解压即可,将bin目录中的程序加入到环境变量
```bash
vim /etc/profile
export PATH="$PATH:/usr/local/bin/mongo/"
```

**每一台上启动config server服务**
`mongod --configsvr --dbpath config/data --port 31000 --logpath config/log/config.log --fork`

**每一台上启动分片服务**
`mongod --shardsvr --replSet shard1 --port 32001 --dbpath shard1/data --logpath shard1/log/shard1.log --fork --journal --profile=1 --slowms=100 --storageEngine wiredTiger
mongod --shardsvr --replSet shard2 --port 32002 --dbpath shard2/data --logpath shard2/log/shard2.log --fork --journal --profile=1 --slowms=100 --storageEngine wiredTiger
mongod --shardsvr --replSet shard3 --port 32003 --dbpath shard3/data --logpath shard3/log/shard3.log --fork --journal --profile=1 --slowms=100 --storageEngine wiredTiger`


**每一台上启动mongos服务**
`mongos --configdb 10.20.49.142:31000,10.20.49.143:31000,10.20.49.144:31000  --port 30000   --logpath  mongos/log/mongos.log --fork
`

**配置副本集**
```
#第一台mongo1
mongo 127.0.0.1:32001
use test;
db.dropDatabase();
db.setProfilingLevel(1); 
use admin;
config = { _id:"shard1", members:[
                     {_id:0,host:"10.20.49.142:32001"},
                     {_id:1,host:"10.20.49.143:32001"},
# 这里没有执行         {_id:2,host:"10.20.49.144:22001",arbiterOnly:true}
                ]
         };
rs.initiate(config); 

#第二台mongo2
mongo 127.0.0.1:32002
use test;
db.dropDatabase();
db.setProfilingLevel(1); 
use admin;
config = { _id:"shard2", members:[
                     {_id:0,host:"10.20.49.144:32002"},
                     {_id:1,host:"10.20.49.143:32002"},
# 这里没有执行         {_id:2,host:"10.20.49.142:32002",arbiterOnly:true}
                ]
         };
rs.initiate(config);

#第三台mongo3
mongo 127.0.0.1:32003
use test;
db.dropDatabase();
db.setProfilingLevel(1); 
use admin;
config = { _id:"shard3", members:[
                     {_id:0,host:"10.20.49.144:32003"},
                     {_id:1,host:"10.20.49.142:32003"},
# 这里没有执行         {_id:2,host:"10.20.49.143:32003",arbiterOnly:true}
                ]
         };
rs.initiate(config);
```


**连接到mongos设置分片**
```
mongo 127.0.0.1:30000
use admin;
db.runCommand( { addshard : "shard1/10.20.49.142:32001,10.20.49.143:32001"});
db.runCommand( { addshard : "shard2/10.20.49.143:32002,10.20.49.144:32002"});
db.runCommand( { addshard : "shard3/10.20.49.142:32003,10.20.49.144:32003"});
```

**添加仲裁节点**
> 这里还需要单独添加仲裁成员的原因：
> 是因为在配置mongod shard角色的时候没有配置仲裁，如果前面启动配置了，这里不需要执行
```
#第一台的仲裁节点在mongo3上
mongo 127.0.0.1:32001
rs.addArb("10.20.49.144:32001");

#第二台的仲裁节点在mongo1上
mongo 127.0.0.1:32002
rs.addArb("10.20.49.142:32002");

#第三台的仲裁节点在mongo2上
mongo 127.0.0.1:32003
rs.addArb("10.20.49.143:32003");
```


**基本命令，查看、修改**
```bash
# 查看谁是primary节点
rs.isMaster()

# 查看节点配置
rs.conf()

# 删除节点
rs.remove("ip:port")

# 修改配置以后
rs.reconfig(config)
```


----------
**备份以及还原**
```bash
#导出所有的数据库
mongodump -h 127.0.0.1 -o /home/qqdz/mongobackup/171010-11

#导出指定数据库
mongodump -h 192.168.1.108 -d dbname -o /home/qqdz/mongobackup/171010-11/some_database

#恢复所有的数据库
mongorestore  /home/qqdz/mongobackup/171010-11

#恢复指定的数据库
mongorestore -d dbname /home/qqdz/mongobackup/171010-11/some_database
```

----------
**相关资料：**
[mongodb的分片和副本集搭建](http://www.ityouknow.com/mongodb/2017/08/05/mongodb-cluster-setup.html)
[分片四](http://www.lanceyan.com/tech/arch/mongodb_shard1.html)
[mongodb备份还原](http://blog.51yip.com/nosql/1573.html)