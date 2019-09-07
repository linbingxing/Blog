# 分布式ID

------

传统的关系型数据库，如MySQL中，数据库本身自带自增主键生成机制，但在分布式环境下，由于分库分表导致数据水平拆分后无法使用单表自增主键，因此我们需要一种全局唯一id生成策略作为分布式主键。

当前业界已经有不少成熟的方案能够解决分布式主键的生成问题，如：UUID、SnoWflake算法（Twitter）、Leaf算法（美团点评）等。



##  UUID

------

UUID(Universally Unique Identifier)的标准型式包含32个16进制数字，以连字号分为五段，形式为8-4-4-4-12的36个字符，示例：`550e8400-e29b-41d4-a716-446655440000`，到目前为止业界一共有5种方式生成UUID，详情见IETF发布的UUID规范 [A Universally Unique IDentifier (UUID) URN Namespace](http://www.ietf.org/rfc/rfc4122.txt)。

UUID具有如下特点：

1. 经由一定的算法机器生成，算法定义了网卡MAC地址、时间戳、名字空间（Namespace）、随机或伪随机数、时序等元素，以及从这些元素生成UUID的算法。UUID的复杂特性在保证了其唯一性的同时，意味着只能由计算机生成。
2. 非人工指定，非人工识别。UUID的复杂性决定了“一般人“不能直接从一个UUID知道哪个对象和它关联。
3. 在特定的范围内重复的可能性极小。

优点：

- 性能非常高：本地生成，没有网络消耗。

缺点：

-  不易于存储：UUID太长，16字节128位，通常以36长度的字符串表示，很多场景不适用。
- 信息不安全：基于MAC地址生成UUID的算法可能会造成MAC地址泄露，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置。
- ID作为主键时在特定的环境会存在一些问题，比如做DB主键的场景下，UUID就非常不适用。

## SnoWflake

------

雪花算法（SnoWflake）是Twitter公布的分布式主键生成算法，也是ShardingSphere默认提供的配置分布式主键生成策略方式。

在ShardingSphere的类路径为：**io.shardingsphere.core.keygen.DefaultKeyGenerator**。

snowflake算法所生成的ID结构是什么样子呢？我们来看看下图：

SnowFlake所生成的ID一共分成四部分：

1. **第一位**：占用1bit，其值始终是0，没有实际作用。
2. **时间戳**：占用41bit，精确到毫秒，总共可以容纳约69 年的时间。
3. **工作机器id**：占用10bit，其中高位5bit是数据中心ID（datacenterId），低位5bit是工作节点ID（workerId），做多可以容纳1024个节点。
4. **序列号**：占用12bit，这个值在同一毫秒同一节点上从0开始不断累加，最多可以累加到4095。

优点：

- 生成ID时不依赖于DB，完全在内存生成，高性能高可用
- ID呈趋势递增，后续插入索引树的时候性能较好。

缺点：依赖于系统时钟的一致性。如果某台机器的系统时钟回拨，有可能造成ID冲突，或者ID乱序。



## Leaf算法

------

对Leaf算法，推荐访问以下链接 

 [Leaf——美团点评分布式ID生成系统](https://tech.meituan.com/2017/04/21/mt-leaf.html)

[Leaf：美团分布式ID生成服务开源](https://tech.meituan.com/2019/03/07/open-source-project-leaf.html)

Github上开源：https://github.com/Meituan-Dianping/Leaf



## Redis计数器

------

使用Redis的 **INCR key**自增计数器，它是 Redis 的原子性自增操作最直观的模式，其原理相当简单：每当某个操作发生时，向 Redis 发送一个 INCR 命令。

优点：

- 性能比数据库好，能满足有序递增。

缺点：

- 由于redis是内存的KV数据库，即使有AOF和RDB，但是依然会存在数据丢失，有可能会造成ID重复。
- 依赖于redis，redis要是不稳定，会影响ID生成。

适用：由于其性能比数据库好，但是有可能会出现ID重复和不稳定，这一块如果可以接受那么就可以使用。也适用于到了某个时间，比如每天都刷新ID，那么这个ID就需要重置，通过(Incr Today)，每天都会从0开始加。

##  Zookeeper

------

通过利用zookeeper的持久顺序节点特性，多个客户端同时创建同一节点，zk可以保证有序的创建，创建成功并返回的path类似于/root/node/0000000001这样的节点，能够看到是顺序有规律的。利用这个特性，我们能够实现基于zk的分布式id生成器。不过，在高并发的条件下，调用zookeeper性能受到影响。

参考资料

[如果再有人问你分布式 ID，这篇文章丢给他](https://juejin.im/post/5bb0217ef265da0ac2567b42)

[分布式 ID 生成方案探讨](https://blog.letiantian.me/microservices/distributed-id.html)

