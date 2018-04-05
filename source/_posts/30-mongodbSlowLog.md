---
title: ELK系列02-ELK分析MongoDB日志
date: 2018-04-05 17:07:29
tags: ELK
categories: 数据分析
---

{% note info %}

线上有部分业务使用`MongoDB`作为存储
目前存在的问题是，并发很高的情况下经常宕掉，所以需要对一些指标进行监控、分析
监控使用`zabbix`，负责监视健康检查、连接数、占用服务器性能等指标
分析使用`ELK`，不多说，let's do it

{% endnote %}

<!-- more -->



### 环境准备:

`mongodb`需要开启`profile`性能分析。
统一将安装包放入`/opt/`下,在`mongo`服务器上只需要部署`filebeat`来负责采集日志
`Filebeat`是轻量级的采集,不负责过滤,启动快,资源消耗小,适合做采集日志工具
直接使用`Filebeat`输出到`ElasticSearch`中也是可以的,数据是原样的,无法做分析
日志过滤和存储通过`Logstash`和`ElasticSearch`来负责,展示通过`Kibana`

### 下载安装包:

需要用到3个组件:

```bash
# filebeat
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.3-linux-x86_64.tar.gz
# Logstash
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.2.3.tar.gz
```

### 配置Filebeat:

`Filebeat`需要配置的不多,只需要确定`log`所在目录,能正常抓取、输出到`Logstash`中

```bash
# cd /opt/filebeat
# cat /opt/filebeat/mongo_fields.yml  
filebeat.prospectors:
- type: log
  paths:
    - /home/qqdz/mongo/shard1/log/shard1.log

output.logstash:
  hosts: ["10.20.33.52:5044"]
  
# 启动
./filebeat -e -c mongo_fields.yml
```



### 配置Logstash:

我这里`Logstash`和`ElasticSearch`在一台上面

`Logstash`的配置复杂一些:

```bash
# cd /opt/logstash
# cat mongodb-pipline.conf
input {
    beats {
        port => "5044"
        client_inactivity_timeout => "86400"
    }
}

filter {
  grok {
       match => ["message","%{TIMESTAMP_ISO8601:timestamp}\s+%{MONGO3_SEVERITY:severity}\s+%{MONGO3_COMPONENT:component}%{SPACE}(?:\[%{DATA:context}\])?\s+%{GREEDYDATA:body}"]
  } 
  if [body] =~ "ms$"  {  
       grok {
             match => ["body",".*\}(\s+%{NUMBER:spend_time:int}ms$)?"]
       }
 }
 date {
   match => [ "timestamp", "ISO8601" ]
   remove_field => [ "timestamp" ]
  }
}

output {
  elasticsearch { 
        hosts => ["localhost:9200"] 
        index=>"mongodb-slow-log-%{+YYYY.MM.dd}"
        }
  #stdout { codec => rubydebug }
}

# 检测配置
logstash-6.2.2/bin/logstash -f mongodb-pipline.conf --config.test_and_exit
# 启动(加automatic以后,修改配置不需要重启Logstash)
logstash-6.2.2/bin/logstash -f mongodb-pipline.conf --config.reload.automatic
```



### 配置ElasticSearch,查看索引:

```bash
# 我截取了关键的几个地方,可以到自动生成了根据Logstash定义的索引
# 可以看到正常输出了
[root@CHJ-Transit02 logstash]# curl 'localhost:9200/_cat/indices?v'
health status index                       pri rep docs.count  store.size pri.store.size
green  open   mongodb-slow-log-2018.03.28   5   1     201952     232.7mb        116.3mb
green  open   mongodb-slow-log-2018.03.30   5   1     163233     192.6mb         96.3mb
green  open   mongodb-slow-log-2018.03.29   5   1     326301     377.6mb        188.7mb
```



### 注意点:

1. 最复杂的地方是`grok`过滤规则的编写,需要大量的练习
2. 官方定义了常规解析,我们还需要根据`mongodb`日志来分割取值




### 链接

```bash
# 这个目录里面存放了很多预定义模式
logstash-6.2.2/vendor/bundle/jruby/2.3.0/gems/logstash-patterns-core-4.1.2/patterns
```

[官方预定义Grok](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns)
[Mongodb预定义](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/mongodb)
[大部分应用的预定义](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)