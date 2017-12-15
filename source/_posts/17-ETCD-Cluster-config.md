---
title: Etcd集群配置
date: 2017-12-15 23:17:47
tags:
- system
categories:
- linux
---

{% note info %}
**etcd**是以实现*共享配置和服务发现*为目的提供一致性的键值存储的分布式数据库。
著名的容器编排系统kubernetes项目使用了etcd。

{% endnote %}



**服务器环境：**

|ip|client_port|peer_port|
|:----:|:----:|:---:|
|10.20.48.107|2379|2380|
|10.20.48.108|2379|2380|
|10.20.48.109|2379|2380|

<!-- more -->


**安装etcd：**
安装比较简单，我们只需要去[官方](https://github.com/coreos/etcd/releases)下载包就可以
+ `wget https://github.com/coreos/etcd/releases/download/v3.1.11/etcd-v3.1.11-linux-amd64.tar.gz`
+ `tar zxvf etcd-v3.1.11-linux-amd64.tar.gz -C /usr/local/`
+  在`/usr/local/` 目录下 `mv etcd-v3.1.11-linux-amd64.tar.gz etcdv3`
+ `ln -s /usr/local/etcdv3/etcd* /usr/local/bin/`
+ 在`/etc/profile`中添加环境变量`export ETCDCTL_API=3`

**配置说明：**
```yaml
name: #节点的名称
initial-advertise-peer-urls: #该成员的对等url列表，用于向集群的其他成员通告（告诉其他成员通过什么url和我通信）
listen-peer-urls:	# 本地监听，接收同等节点请求的端口
listen-client-urls:  # 本地监听，接受客户端请求的端口
advertise-client-urls: # 该成员的客户端url的列表，用于向所有人通告（告知所有人client访问我的哪儿）
initial-cluster-token:  # 集群唯一令牌，识别不同的集群
initial-cluster: # 集群的成员，name是前面指定name
initial-cluster-state: # 初始集群的状态，有“new,existing”
data-dir: # 数据存储的目录
```

**配置文件：**
我们通过配置文件的方式启动
etcd1：
```yaml
name: "etcd1"
initial-advertise-peer-urls: "http://10.20.48.107:2380"
listen-peer-urls: "http://10.20.48.107:2380"
listen-client-urls: "http://10.20.48.107:2379,http://127.0.0.1:2379"
advertise-client-urls: "http://10.20.48.107:2379"
initial-cluster-token: "heyhey-etcd"
initial-cluster: "etcd1=http://10.20.48.107:2380,etcd2=http://10.20.48.108:2380,etcd3=http://10.20.48.109:2380"
initial-cluster-state: "new"
data-dir: "/home/qqdz/etcd/etcd1"
```

etcd2：
```yaml
name: "etcd2"
initial-advertise-peer-urls: "http://10.20.48.108:2380"
listen-peer-urls: "http://10.20.48.108:2380"
listen-client-urls: "http://10.20.48.108:2379,http://127.0.0.1:2379"
advertise-client-urls: "http://10.20.48.108:2379"
initial-cluster-token: "heyhey-etcd"
initial-cluster: "etcd1=http://10.20.48.107:2380,etcd2=http://10.20.48.108:2380,etcd3=http://10.20.48.109:2380"
initial-cluster-state: "new"
data-dir: "/home/qqdz/etcd/etcd2"
```

etcd3：
```yaml
name: "etcd3"
initial-advertise-peer-urls: "http://10.20.48.109:2380"
listen-peer-urls: "http://10.20.48.109:2380"
listen-client-urls: "http://10.20.48.109:2379,http://127.0.0.1:2379"
advertise-client-urls: "http://10.20.48.109:2379"
initial-cluster-token: "heyhey-etcd"
initial-cluster: "etcd1=http://10.20.48.107:2380,etcd2=http://10.20.48.108:2380,etcd3=http://10.20.48.109:2380"
initial-cluster-state: "new"
data-dir: "/home/qqdz/etcd/etcd3"
```
**启动脚本：**
```bash
#!/bin/sh

startwork()
{
        nohup /usr/local/bin/etcd --config-file /home/qqdz/etcd/etcd.yml  & 

        echo "ps x | grep \"etcd\""
        ps x|grep "etcd"   
}

stopwork()
{
        SERVERLIST='etcd'

        for serv in $SERVERLIST
        do
                echo "stop $serv"
                ps aux|grep "/$serv"|sed -e "/grep/d"|awk '{print $2}'|xargs kill 2&>/dev/null
                sleep 1
        done

    while test -f run.sh
    do
        count=`ps x|grep "etcd-c"|sed -e '/grep/d'|wc -l`
        echo "running server:"$count
        if [ $count -eq 0 ]; then
            break
        fi
        sleep 1
    done
}

echo "-------------------start-----------------------"

case $1 in
stop)
    stopwork
;;
start)
    startwork
;;
*)
    stopwork
    sleep 1
    startwork
;;
esac

echo "-------------------end-----------------------"
```



**常用命令：**
获取etcd所有的keys
`ETCDCTL_API=3 etcdctl get "" --prefix=true`
`ETCDCTL_API=3 etcdctl get "" --from-key`

查看集群状态
在`.bash_profile`中添加下面语句，这样每次登录都会自动加载变量
`ENDPOINTS=10.20.48.107:2379,10.20.48.108:2379,10.20.48.109:2379`
`export ENDPOINTS`

通过下面命令查看集群状态：
`etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status`
```bash
[root@localhost ~]$ etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status     
+-------------------+------------------+---------+---------+-----------+-----------+------------+
|     ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+-------------------+------------------+---------+---------+-----------+-----------+------------+
| 10.20.48.107:2379 | 40c1a9f933c13188 | 3.1.11  | 33 kB   | true      |       176 |         41 |
| 10.20.48.108:2379 | c0598886ee4f3484 | 3.1.11  | 33 kB   | false     |       176 |         41 |
| 10.20.48.109:2379 | 3837344de92ee822 | 3.1.11  | 33 kB   | false     |       176 |         41 |
+-------------------+------------------+---------+---------+-----------+-----------+------------+
```

通过下面命令查看健康状态：
`etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint health`
```bash
[root@localhost ~]$ etcdctl --endpoints=$ENDPOINTS endpoint health                
10.20.48.107:2379 is healthy: successfully committed proposal: took = 1.249516ms
10.20.48.109:2379 is healthy: successfully committed proposal: took = 1.415811ms
10.20.48.108:2379 is healthy: successfully committed proposal: took = 1.487734ms
```

启动Etcd网关：
`etcd gateway start --endpoints=10.20.48.107:2379,10.20.48.108:2379,10.20.48.109:2379 --listen-addr="127.0.0.1:23790"`

----------
资料：
[Etcd交互](https://coreos.com/etcd/docs/latest/dev-guide/interacting_v3.html)
[Etcd--config-file配置模板](https://github.com/coreos/etcd/blob/master/etcd.conf.yml.sample)
[Etcd配置说明](https://coreos.com/etcd/docs/latest/op-guide/configuration.html)
