#  redis

1. 基本类型
2. 存储结构
3. 使用场景
4. 高可用部署

##  一、基本类型及数据结构

1. 字符串 String 

   可以存储字节串byte string、整数、浮点数

2. 列表  List quicklist -> ziplist

   常见命令：lpush,rpush、lpop、rpop、lrange ,ltrim、blpop、brpop

3. 哈希 Hash hashtable-ziplist

   

4. 集合 Set  hashtable

   常见命令：sadd、smembers、scard、sismember

5. 有序集合 SortSet skiplist(跳表实现) hashtable

过期时间实现原理

消极处理

积极处理：周期性从设置过期时间的key中间选择一部分的key进行删除



##  二、持久化

####  快照 snapshotting  RDB

save second  keys

缺点：数据容易丢失

####  只追加 append-only-file (AOF)

```
appendonly no

appendfilename "appendonly.aof"

appendfsync everysec 每秒进行同步，显式地将多个写命令同步到硬盘 

appendfsync always  每次写入

appendfsync no 让操作系统来决定何时进行同步

缺点：aof文件会很大

重写和压缩aof文件

auto-aof-rewrite-percentage 100  比上次重写的文件大小大于100%
auto-aof-rewrite-min-size 文件大小
 
重写缓存概念
```

####   redis的内存回收策略

LRU

maxmemory-policy noeviction 



allkeys-lru  最少使用的数据淘汰

allkeys-random

valatile-random/lru/ttl  即将过期的数据进行淘汰



####  Redis单线程为什么性能很高

内存和网络带宽限制

多路复用机制（需要了解）

###  lua脚本在redis使用

主要保证原子性操作

多个命令操作，影响网络请求

pipeline管道模型，多个命令同时执行，减少网络开销，满足原子性

复用性处理

调用方式: redis.call('set','test','23432');

eval script numberkeys key ... arg ...

####  实现限流

local  key="ratelimit:"..KEYS[1]

local limit = tonumber(ARGV[1])

local expireTIme = ARGV[1]

local times = redis.call('incr',key)

if times == 1 then

   redis.call('expire',key,expireTime)

end

if times > limit then

  return 0

end

return 1





script load "脚本"

生成脚本摘要

evalsha  





### redis集群

*  单机版
* 哨兵模式
* 集群模式
* 主从模式(RDB文件复制到从服务器)
   slaveof 主服务器ip 端口

 同步:    全量复制（bgsave)，增量复制（命令发送），无磁盘复制

哨兵：

1、监控master和slave是否正常运行

 2、当master出现故障的时候，从slave选举一个新的master

3、哨兵内部选举机制raft协议

4、哨兵会通过发布/订阅方式监听master



集群处理：分片处理，插槽

一致性hash->codis(豌豆荚插件)

gossip协议的无中心化节点的集群

虚拟槽的概念（0-16383 slot)

CRC16(key) %16382 = 槽位

如何让key落在同一槽位上？？？

HashTag--->

user:{user1)}:id

user:{user1}:name

user:{user1}:sex

分片迁移？？动态扩容

slot的分配，各个节点取部分的数据来均衡分配

面临两个问题：数据迁移、槽位迁移？



 MasterA                          MasterB

MIGRANING                    IMPORTING

表示slot正在迁移             表示slot 正在迁入



1、如果客户端访问的key还没迁移出去，则正常处理key

2、如果



# [Docker Compose 部署Redis Sentinel集群](https://www.guonanjun.com/272.html)

