---
title: redis配置详解
date: 2018-04-19 14:46:18
tags: redis
categories: db
---


{% note info %}

`redis`中有很多配置选项，下面我们来针对配置做一个详细说明

{% endnote %}

<!-- more -->


```bash
#是否在后台运行
daemonize yes
# 当redis在后台运行的时候，默认会把pid文件放在/var/run/redis.pid，你可以配置到其他地址。
# 当运行多个redis服务时，需要指定不同的pid文件和端口
pidfile /home/qqdz/redis/pid/redis_6101.pid
# 指定运行的端口
port 6101
# 此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度
tcp-backlog 511
# 设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接
# 0是关闭此设置, 永不超时
timeout 0
# 如果值非0，单位是秒，表示将周期性的使用SO_KEEPALIVE检测客户端是否还处于健康状态，避免服务器一直阻塞，官方给出的建议值是60S。
# 0是永不检测
tcp-keepalive 0
# Redis总共支持四个级别：debug、verbose、notice、warning。
# Notice：普通的verbose，常用于生产环境；
loglevel notice
# 日志的存储路径
logfile "log_6101.log"
# 数据库的数量是可以配置的
# 可用的数据库数，默认值为16，默认数据库为0，数据库范围在0-（database-1）之间
databases 16
# 指出在多长时间内，有多少次更新操作，就将数据同步到数据文件rdb。
# 相当于条件触发抓取快照，这个可以多个条件配合
save 900 1		# 900秒内至少有1个key被改变
save 300 10		# 300秒内至少有300个key被改变
save 60 10000	# 60秒内至少有10000个key被改变
# 当持久化出现错误之后，是否继续提供写服务
stop-writes-on-bgsave-error yes
# 存储至本地数据库时（持久化到rdb文件）是否压缩数据
rdbcompression no
# 读取和写入的时候是否支持CRC64校验
rdbchecksum no
# 本地持久化数据库文件名，默认值为dump.rdb
dbfilename rdb_6101.rdb
# 数据库镜像备份的文件放置的路径。
# 注意这里必须制定一个目录而不是文件
dir /home/qqdz/redis/store

# 当slave服务器和master服务器失去连接后，或者当数据正在复制传输的时候，如果此参数值设置“yes”，slave服务器可以继续接受客户端的请求，否则，会返回给请求的客户端如下信息“SYNC with master in progress”
slave-serve-stale-data yes

# 是否允许slave服务器节点只提供读服务
slave-read-only yes

# 无磁盘diskless：master端直接将RDB file传到slave socket，不需要与disk进行交互
# 默认不使用diskless同步方式 
repl-diskless-sync no

# 一个RDB文件从master端传到slave端，分为两种情况： 
# 1、支持disk：master端将RDB file写到disk，稍后再传送到slave端；
# 2、无磁盘diskless：master端直接将RDB file传到slave socket，不需要与disk进行交互。 
# 无磁盘diskless方式在进行数据传递之前会有一个时间的延迟，以便slave端能够进行到待传送的目标队列中，这个时间默认是5秒
repl-diskless-sync-delay 5

# 是否启用TCP_NODELAY，如果启用则会使用少量的TCP包和带宽去进行数据传输到slave端，当然速度会比较慢；如果不启用则传输速度比较快，但是会占用比较多的带宽
repl-disable-tcp-nodelay no

# 指定slave的优先级。在不只1个slave存在的部署环境下，当master宕机时，Redis Sentinel会将priority值最小的slave提升为master。需要注意的是，若该配置项为0，则对应的slave永远不会自动提升为master
slave-priority 100

# 开启append only 模式之后，redis 会把所接收到的每一次写操作请求都追加到appendonly.aof 文件中，当redis 重新启动时，会从该文件恢复出之前的状态。但是这样会造成appendonly.aof 文件过大，所以redis 还支持了BGREWRITEAOF 指令，对appendonly.aof 进行重新整理。默认是不开启的。
#appendonly yes

# aof文件名
#appendfilename "aof_6101.aof"

# no: 不进行同步，系统去操作 . Faster.
# always: always表示每次有写操作都进行同步. Slow, Safest.
# everysec: 表示对写操作进行累积，每秒同步一次. Compromise.
# 默认是"everysec"，按照速度和安全折中这是最好的。
#appendfsync everysec

# 指定是否在后台aof文件rewrite期间调用fsync，默认为no，表示要调用fsync（无论后台是否有子进程在刷盘）。Redis在后台写RDB文件或重写afo文件期间会存在大量磁盘IO，此时，在某些linux系统中，调用fsync可能会阻塞。
no-appendfsync-on-rewrite no

# 指定Redis重写aof文件的条件，默认为100，表示与上次rewrite的aof文件大小相比，当前aof文件增长量超过上次afo文件大小的100%时，就会触发background rewrite。若配置为0，则会禁用自动rewrite
auto-aof-rewrite-percentage 100
# 指定触发rewrite的aof文件大小。若aof文件小于该值，即使当前文件的增量比例达到auto-aof-rewrite-percentage的配置值，也不会触发自动rewrite。即这两个配置项同时满足时，才会触发rewrite。
auto-aof-rewrite-min-size 64mb

#  是否加载不完整的aof文件来进行启动
aof-load-truncated yes

# 一个Lua脚本最长的执行时间，单位为毫秒，如果为0或负数表示无限执行时间，默认为5000
lua-time-limit 5000

# redis的slow log是一个系统OS进行的记录查询，它是超过了指定的执行时间的。执行时间不包括类似与client进行交互或发送回复等I/O操作，它只是实际执行指令的时间。 
# 有2个参数可以配置，一个用来告诉redis执行时间，这个时间是微秒级的（1秒=1000000微秒），这是为了不遗漏命令。另一个参数是设置slowlog的长度，当一个新的命令被记录时，最旧的命令将会从命令记录队列中移除。 
slowlog-log-slower-than 10000
slowlog-max-len 128

# 延迟监控，用于记录等于或超过了指定时间的操作，默认是关闭状态，即值为0
latency-monitor-threshold 0
# 事件通知，默认不启用
notify-keyspace-events ""

# 当条目数量较少且最大不会超过给定阀值时，哈希编码将使用一个很高效的内存数据结构，阀值由以下参数来进行配置
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# 与哈希类似，少量的lists也会通过一个指定的方式去编码从而节省更多的空间，它的阀值通过以下参数来进行配置。 
list-max-ziplist-entries 512
list-max-ziplist-value 64

# 集合sets在一种特殊的情况时有指定的编码方式，这种情况是集合由一组10进制的64位有符号整数范围内的数字组成的情况。以下选项可以设置集合使用这种特殊编码方式的size限制
set-max-intset-entries 512

# 与哈希和列表类似，有序集合也会使用一种特殊的编码方式来节省空间，这种特殊的编码方式只用于这个有序集合的长度和元素均低于以下参数设置的值时。
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# 设置HyeperLogLog的字节数限制，这个值通常在0~15000之间，默认为3000，基本不超过16000 
hll-sparse-max-bytes 3000

# redis将会在每秒中抽出10毫秒来对主字典进行重新散列化处理，这有助于尽可能的释放内存
activerehashing yes

# 如果达到hard limit那client将会立即失连。 
# 如果达到soft limit那client将会在soft seconds秒之后失连。 
# 参数soft limit < hard limit。 
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# redis使用一个内部程序来处理后台任务，例如关闭超时的client连接，清除过期的key等等。它并不会同时处理所有的任务，redis通过指定的hz参数去检查和执行任务。 
# hz默认设为10，提高它的值将会占用更多的cpu，当然相应的redis将会更快的处理同时到期的许多key，以及更精确的去处理超时。 
# hz的取值范围是1~500，通常不建议超过100，只有在请求延时非常低的情况下可以将值提升到100。 
hz 10

# 当一个子进程要改写AOF文件，如果以下选项启用，那文件将会在每产生32MB数据时进行同步，这样提交增量文件到磁盘时可以避免出现比较大的延迟。 
aof-rewrite-incremental-fsync yes

# 关闭保护模式
protected-mode no
```

动态关闭保护模式
```powershell
redis-cli -p 6500 CONFIG SET protected-mode no
```