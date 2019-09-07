# Redis知识总结(三)-集群部署

在现实的生产环境中，我们不可能只启动一台Redis实例，所以就需要了解Redis的集群部署，我们知道Redis的部署可以通过以下几种模式。

- 主从模式(RDB文件复制到从服务器)
- 哨兵模式
- 集群模式

------

## 主从模式

### 1. **服务架构**

   ![主从服务架构](C:\Users\PC4\Desktop\TIM截图20190504004640.png)

### 2. **实现原理**

   Redis的主从模式，主要有三种复制模式，全量复制、增量复制、无磁盘复制。

   全量复制：一般在初始化的时候，比如在新加入从节点的时候，主节点会把数据全量复制到从节点。

   增量复制：在全量复制完以后，主节点的更新操作就会以增量方式进行处理。

   无磁盘复制：主节点不在生成RDB文件发送到从节点，而是直接通过内存数据进行发送，版本2.8以后提供，配置参数repl-diskless-sync

   主从复制功能的详细步骤可以分为7个步骤：

   * 设置主服务器的地址和端口
   * 建立套接字连接
   * 发送PING命令
   * 身份验证
   * 发送端口信息
   * 全量同步
   * 增量同步-命令传播

   ![同步流程](C:\Users\PC4\Desktop\TIM截图20190504174231.png)

   全量同步的步骤如下：

   * 主节点收到从服务器的全量重同步请求时，主服务器便开始执行bgsave命令，同时用一个缓冲区记录从现在开始执行的所有写命令。
   * 当主服务器的bgsave命令执行完毕后，会将生成的RDB文件发送给从服务器。从服务器接收到RDB文件时，会将数据文件保存到硬盘，然后加载到内存中。
   * 主服务器将缓冲区所有缓存的命令发送到从服务器，从服务器接收并执行这些命令，将从服务器同步至主服务器相同的状态。

   增量同步：当全量同步完以后，redis后面开始进行增量同步。那么怎么进行增量同步呢? 其实也就是全量同步的第三步骤，将主服务器的操作命令发送到从服务器执行命令。那如何能保证从服务器完整接收并执行命令呢? 需要先了解一下几个概念：

   * 运行ID（每个Redis实例都会生成一个运行id)
      当从服务器对主服务器进行初次复制时，主服务器会发送自己的运行ID给从服务器。
      当从服务器断线重连时，会将之前主服务器的运行ID发送给当前连接的主服务器。这时候会出现下面两种情况

     1. 运行ID和主服务器不一致，说明之前连接的主服务器与本次连接不同，开始执行全量重同步操作。
     2. 运行ID和主服务器一致，主服务器可以尝试执行部分重同步操作。

   * 复制偏移量

     从主服务器的复制信息可以看到从服务器slave0和slave1都有一个参数offset，这个参数就是从服务器的复制偏移量。master_repl_offset这个参数就是主服务器的偏移量，slave_repl_offset是从服务器的偏移量。如下图。通过两者的对比，就可以知道命令是否丢失，丢失则补发复制偏移量相差的字节命令。

     ![](C:\Users\PC4\Desktop\TIM截图20190505091431.png)

     ![TIM截图20190505092504](C:\Users\PC4\Desktop\TIM截图20190505092504.png)

   * 复制缓冲区

   ​          在命令传播阶段，主节点除了将写命令发送给从节点，还会发送一份给复制积压缓冲区。复制缓冲区里面会保存着一部分最传播的写命令和每个字节相应的复制偏移量。因为复制缓冲区的大小是有限制的，所以保存的数据也是有限制的。如果从服务器与主服务器的复制偏移量相差的数据大于复制缓冲去存储的数据时，同样不会执行部分重同步，而会去执行全量同步。

   ​     

### 3. **部署配置**

   1. 环境说明

      ```
      系统：CentOS-7-x86_64
      redis版本：5.0.4
      ip：192.168.175.101
      redis节点：
      ```

      ```properties
      	192.168.175.101：6379 master
          192.168.175.101：6380 slave
          192.168.175.101：6381 slave
      ```

   2. 修改配置，并启动

      将redis.conf复制两份，修改port、pidfile、slaveof，其他保持默认

      ```shell
      cp redis.conf redis_6380.conf
      cp redis.conf redis_6381.conf
      
      #修改port、pidfile、添加 slaveof
      vi redis-6380.conf
          port 6380
          protected-mode no
          masterauth 123456
          requirepass 123456
          pidfile /var/run/redis-6380.pid
          slaveof 192.168.175.101 6379
      
      vi redis-6381.conf
          port 6381
          protected-mode no
          masterauth 123456
          requirepass 123456
          pidfile /var/run/redis-6381.pid
          slaveof 192.168.175.101 6379
          
      #启动master和2个slave
      ./src/redis-server redis.conf 
      ./src/redis-server redis-6380.conf 
      ./src/redis-server redis-6381.conf
      ```

   3. 查看是否部署成功

      ```shell
      #打开1个shell窗口，登录主节点
      [root@centos101 redis-5.0.4]# ./src/redis-cli -p 6379
      127.0.0.1:6379> info replication
      # Replication
      role:master
      connected_slaves:2
      slave0:ip=127.0.0.1,port=6380,state=online,offset=84,lag=1
      slave1:ip=127.0.0.1,port=6381,state=online,offset=84,lag=0
      master_replid:083a4365862bb1bf32fcfae8f3c678d30b07b821
      master_replid2:0000000000000000000000000000000000000000
      master_repl_offset:84
      second_repl_offset:-1
      repl_backlog_active:1
      repl_backlog_size:1048576
      repl_backlog_first_byte_offset:1
      repl_backlog_histlen:84
      # 可以看到当前节点为主节点，有两个子节点
      
      #我们在看一下子节点的信息，可以看到子节点有对应主节点的ip、端口
      [root@centos101 redis-5.0.4]# ./src/redis-cli -p 6380
      127.0.0.1:6380> info replication
      # Replication
      role:slave
      master_host:192.168.175.101
      master_port:6379
      master_link_status:up
      master_last_io_seconds_ago:8
      master_sync_in_progress:0
      slave_repl_offset:4438
      slave_priority:100
      slave_read_only:1
      connected_slaves:0
      master_replid:083a4365862bb1bf32fcfae8f3c678d30b07b821
      master_replid2:0000000000000000000000000000000000000000
      master_repl_offset:4438
      second_repl_offset:-1
      repl_backlog_active:1
      repl_backlog_size:1048576
      repl_backlog_first_byte_offset:1
      repl_backlog_histlen:4438
      ```

   4. 测试一下

      现在主服务器6379输入命令，可以看到6380可以获取到数据

      ```shell
      [root@centos101 redis-5.0.4]# ./src/redis-cli -p 6379
      127.0.0.1:6379> set test 123456
      OK
      
      [root@centos101 redis-5.0.4]#  ./src/redis-cli -p 6380
      127.0.0.1:6380> get test
      "123456"
      
      #尝试在6380设置值，可以看到报错，只能读不能写
      [root@centos101 redis-5.0.4]#  ./src/redis-cli -p 6380
      127.0.0.1:6380> set test3 123455
      (error) READONLY You can't write against a read only replica.
      
      #测试主节点如何复制数据到从节点，可以看主节点的更新操作命令会同步到从节点
      127.0.0.1:6380> replconf listening-port 6379
      OK
      127.0.0.1:6380> sync
      Entering replica output mode...  (press Ctrl-C to quit)
      SYNC with master, discarding 226 bytes of bulk transfer...
      SYNC done. Logging commands from master.
      "set","test5","123123"
      "PING"
      "set","ggg","123123"
      ```

5. 关于主从复制的几点问题

   * 了解Redis是如何保证主从服务器一致处于连接状态以及命令是否丢失？
   * 如何确保主服务器进行增量同步时，不影响性能？

6. 配置相关参数

   ```properties
   # slaveof <masterip> <masterport>
   slaveof 192.168.175.101 6379
   # masterauth <主服务器Redis密码>
   masterauth 123456
   
   # 当slave丢失master或者同步正在进行时，如果发生对slave的服务请求
   # yes则slave依然正常提供服务
   # no则slave返回client错误："SYNC with master in progress"
   slave-serve-stale-data yes
   
   # 指定slave是否只读
   slave-read-only yes
   
   # 无硬盘复制功能
   repl-diskless-sync no
   
   # 无硬盘复制功能间隔时间
   repl-diskless-sync-delay 5
   
   # 从服务器发送PING命令给主服务器的周期
   # repl-ping-slave-period 10
   
   # 超时时间
   # repl-timeout 60
   
   # 是否禁用socket的NO_DELAY选项
   repl-disable-tcp-nodelay no
   
   # 设置主从复制容量大小，这个backlog 是一个用来在 slaves 被断开连接时存放 slave 数据的 buffer
   # repl-backlog-size 1mb
   
   # master 不再连接 slave时backlog的存活时间。
   # repl-backlog-ttl 3600
   
   # slave的优先级
   slave-priority 100
   
   # 未达到下面两个条件时，写操作就不会被执行
   # 最少包含的从服务器
   # min-slaves-to-write 3
   # 延迟值
   # min-slaves-max-lag 0
   
   #设置缓冲区的大小
   repl-backlog-size 1mb
   
   # repl-backlog-ttl 3600
   
   ```

## 哨兵模式

 ### 1、服务架构



###  2、实现原理



### 3、搭建环境



### 4、测试





