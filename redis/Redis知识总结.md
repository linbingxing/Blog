#   Redis知识总结

Redis的全称是：Remote Dictionary Server

Redis是一个开源（BSD许可），内存数据结构存储，可以用作数据库，缓存和消息代理。它支持数据结构，如字符串，散列，列表，集合，带有范围查询的排序集，位图，超级日志，具有半径查询和流的地理空间索引。Redis具有内置复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并通过Redis Sentinel提供高可用性和使用Redis Cluster自动分区。

## 目录

- 基本类型及数据结构
- Redis的过期策略、内存淘汰策略
- 数据如何持久化
- redis的内存回收策略
- redis如何使用lua脚本
- redis集群部署
- 分布式锁实现

##  一、基本类型及数据结构

### String字符串

可以存储字符串、整数、浮点数、JSON、XML、二进制等，最大不能超过521M，可变的动态字符串。

**1、常用命令**：set、get、mset、mget、setex、setnx、incr、decr，具体使用方法网上很多，不多详述。

**2、内部编码**：

​     在 Redis 中字符串类型的内部编码有 3 种：

- int：8个字节的长整型
- embstr：小于等于39个字节的字符串
- raw：大于39个字节的字符串

**3、使用场景**：

1.  计数器，利用incr、decr等命令实现计算器
2. 共享session
3. 限流、限速

**4、数据结构：**

​      SDS （Simple Dynamic String，简单动态字符串）是 Redis 底层所使用的字符串表示， 几乎所有的 Redis 模块中都用了 sds。

​     当字符串对象保存的是字符串时， 它包含的才是 sds 值， 否则的话， 它就是一个 `long` 类型的值。

​    sds 既可高效地实现追加和长度计算， 同时是二进制安全的。

​    后续详解。

   参考资料：

​        [Redis内部数据结构详解(2)——sds](http://zhangtielei.com/posts/blog-redis-sds.html)

​        [简单动态字符串](https://redisbook.readthedocs.io/en/latest/internal-datastruct/sds.html)

### List列表

​      一个列表可以有序地存储多个字符串，并且列表里的元素是可以重复的。可以对列表的两端进行插入或者弹出元素操作。

​     特点如下：

   *   列表中所有的元素都是有序的，所以它们是可以通过索引获取的，也就是 lindex 命令。并且在 Redis 中列表类型的索引是从 0 开始的。
   *   列表中的元素是可以重复的，也就是说在 Redis 列表类型中，可以保存同名元素。

1. **常用命令**

   * 从右边/左边插入元素  ``` rpush/lpush key value [value ...]```
   * 向某个元素前或者后插入元素  ``` linsert key BEFORE|AFTER pivot value```
   * 获取指定范围内的元素列表 ```lrange key start stop```
   * 获取列表中指定索引下标的元素 ```lindex key index ```
   * 从列表右侧/左侧弹出元素 ```rpop/lpop key ```
   * 删除指定元素 ```lrem key count value```
   * 按照索引范围修剪列表 ```ltrim key start stop```
   * 阻塞操作 ,阻塞等待timeout秒弹出元素 ```blpop/brpop key timeout```

2. **数据结构**

   列表中的内部数据结构有两种，它们分别是：

   * ziplist（压缩列表）：当列表中元素个数小于 512（默认）个，并且列表中每个元素的值都小于 64（默认）个字节时，Redis 会选择用 ziplist 来作为列表的内部实现以减少内存的使用。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。相关参数修改：list-max-ziplist-entried（元素个数）、list-max-ziplist-value(元素值)。
   * quicklist（快速列表）：当列表数据无法满足ziplist条件时，Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

3. **使用场景**

   - 消息队列

4. **其他**

### Set集合

​      set集合与列表一样，也是可以保存多个字符串

​      特点如下：

   - set 中的元素是不可以重复的，而 list 是可以保存重复元素的。
   - set 中的元素是无序的，而 list 中的元素是有序的。
   - set 中的元素不能通过索引下标获取元素，而 list 中的元素则可以通过索引下标获取元素。
   - 多个set可以取交集、并集、差集。

1.  **常用命令**    

   * 添加元素 ```sadd key member [member...]```

   * 删除元素 ```srem key member [member...]```

   * 计算元素个数 ```scard key ```

   * 判读元素是否在集合中 ```sismember key member```

   * 随机从 set 中返回指定个数元素 ```srandmember key [count]```

   * 从集合中随机弹出元素 ```spop key [count]``

   * 获取所有元素 ```smember key```

   * 集合的交集 ```sinter key [key ...]```

   * 集合的并集 ```sunion key [key ...]```

   * 集合的差集 ```sdiff key [key ...]```

   * 将集合的交集、并集、差集的结果保存

     ```shell
     sinterstore destination key [key ...]
     sunionstore destination key [key ...]
     sdiffstore destination key [key ...]
     ```

2. **数据结构**

   集合的内部数据结构有两种：

   * intset(整数集合)：当集合中的元素都是整数，并且集合中的元素个数小于 512 个时，Redis 会选用 intset 作为底层内部实现。set-max-intset-entries 参数来设置上述中的默认参数。
   * hashtable(哈希表)：当上述条件不满足时，Redis 会采用 hashtable 作为底层实现。

3. **使用场景**

   * 标签（tag）

     一个用户对娱乐、体育比较感兴趣，另一个可能对新闻感兴
     趣，这些兴趣就是标签，有了这些数据就可以得到同一标签的人，以及用户的共同爱好的标签，
     这些数据对于用户体验以及曾强用户粘度比较重要。

     

4. 其他

### Hash散列

​      redis的哈希跟JAVA语言的HashMap类似，都是键值对结构。使用二维结构，第一维是数组，第二维是链表，hash的内容key和value存放在链表中，数组里存放的是链表的头指针。通过key查找元素时，先计算key的hashcode，然后用hashcode对数组的长度进行取模定位到链表的表头，再对链表进行遍历获取到相应的value值，链表的作用就是用来将产生了「hash碰撞」的元素串起来。

![hash存储结构](https://user-gold-cdn.xitu.io/2018/7/23/164c4dcd14c00534?imageslim)

1. **常用命令**

   * 设置值 ```hset key field value```
   * 获取值 ```hget key field```
   * 删除值 ```hdel key  field [field ...]```
   * 计算field个数 ```hlen key ```
   * 批量设置值 ``hmset key field value [field value...]``
   * 批量获取值 ```hmget key field```
   * 判断field是否存在 ```hexists key field```
   * 获取所有field ```hkeys key```
   * 获取所有value ``` hvals key ```
   * 获取所有field -value  ```hgetall  key```
   * 计数  ```hincrby key field increment  hincrbyfloat key field increment```

2. **数据结构**

   Redis 哈希类型的数据结构类型包含两种：

   * ziplist（压缩列表）：当哈希类型中元素个数小于 hash-max-ziplist-entries 配置（默认 512 个），同时所有值都小于 hash-max-ziplist-value 配置（默认 64 字节）时，Redis 会使用 ziplist 作为哈希的内部实现。
   * hashtable（哈希表）：当上述条件不满足时，Redis 则会采用 hashtable 作为哈希的内部实现。

3. **使用场景**

   * 存储对象信息，用户信息、登录信息

4. **其他**

### Zset有序集合

​      Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数(score)却可以重复。

1. **常用命令**

   * 设置值  ```zadd key [NX|XX] [CH] [INCR] score member [score member ...]```
   * 计算成员个数 ```zcard key ```
   * 计算成员分数 ```zscore  key ```  member
   * 计算成员的排名 ```zrank/zrevrank  key member ```
   * 删除元素  ```zrem key member [member ...]```
   * 增加元素分数 ``` zincrby key increment member ```
   * 返回指定排名范围的元素 ```zrange/zrevrange  key start stop [WITHSCORES]```
   * 返回指定分数范围的元素 ``` zrangebyscore/zrevrangebyscore key min max [WITHSCORES] [LIMIT offset count]```
   * 返回指定分数范围元素个数 ``` zcount key min max```
   * 删除指定排名内的升序元素 ``` zremrangebyrank key start stop ```
   * 删除指定分数范围元素 ```zremrangebyscore key min max ```

2. **数据结构**

   有序集合内部的数据结构类型包含两种：

   * ziplist(压缩列表)：当有序集合的元素个数小于 128 个(默认设置)，同时每个元素的值都小于 64 字节(默认设置)，Redis 会采用 ziplist 作为有序集合的内部实现，通过以下参数设置：zset-max-ziplist-entries 和 zset-max-ziplist-value。
   * skiplist(跳跃表)：当上述条件不满足时，Redis 会采用 skiplist 作为内部编码。

3. **使用场景**     

   * 排行榜
   * 

4. **其他**

###  Bitmaps 位图

​     位图不是实际的数据类型，而是在String 字符串类型上定义的一组面向位的操作。位图的最大优势之一是它们在存储信息时通常可以节省大量空间。例如，在不同用户由增量用户ID表示的系统中，可以使用仅512 MB的内存记住40亿用户的单个位信息（例如，知道用户是否想要接收新闻通讯）。

1. **常用命令**

   * 对 `key` 所储存的字符串值，设置或清除指定偏移量上的位(bit) ``` SETBIT key offset value```
   * 对 `key` 所储存的字符串值，获取指定偏移量上的位(bit)  ``` GETBIT key offset```

2. **使用场景**

   * 用户在线状态

   * 统计活跃用户

   * 用户签到

3. **参考资料**

   [BitMap](https://juejin.im/post/5a7dcad0f265da4e6f17d942#heading-21)

### HyperLogLog 

​	Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

1. **常用命令**

   * 添加指定元素到 HyperLogLog 中  ``` pfadd key value ```
   * 返回给定 HyperLogLog 的基数估算值 ``` pfcount key ```
   * 将多个 HyperLogLog 合并为一个 HyperLogLog ``` pfmerge destkey sourcekey [key ...]```

2.  **使用场景**

   一般可以bitmap和hyperloglog配合使用，bitmap标识哪些用户活跃，hyperloglog计数。

   * 统计注册 IP 数
   * 统计每日访问 IP 数
   * 统计页面实时 UV 数
   * 统计在线用户数
   * 统计用户每天搜索不同词条的个数

3. 其他

##  二、Redis的过期策略、内存淘汰策略

​      我们都知道，Redis是key-value数据库，我们可以设置Redis中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了，Redis如何处理？

​     Redis keys的过期策略有两种，一种是消极处理，一种是积极处理。

   -  消极处理：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
   -  积极处理：
      * 每秒执行10次，每次随机获取20个设置过期的key
      * 清除其中已过期的key；
      * 当超过25%的key过期，则重复第一步。

**如何在复制链接和AOF文件处理过期？**

​    官网上解释到，为了保证唯一性，对于一个过期的key操作会到AOF然后同步到子节点，子节点虽然不进行操作过期时间，但是仍然保留对于key的过期机制，避免后面主节点挂了，从节点无法处理。

 **Redis的内存淘汰策略**

Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。

- noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
- allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
- volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
- volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
- volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

配置参数：maxmemory-policy

## 三、数据如何持久化

