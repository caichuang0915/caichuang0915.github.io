---
title: redis
tags:
  - redis
---



#### redis持久化策略

redis持久化策略主要有RDB和AOF两种。

##### RDB

- RDB：在指定的时间间隔能对数据进行快照储存

触发方式

1. 手动触发

SAVE 和 BGSAVE 两个命令都可以手动触发生产RDB文件

 - SAVE: 阻塞当前redis服务，应该禁止该命令。  

 - BGSAVE: 主进程fork出一个子进程进行rdb操作，主线程继续响应其它的操作，为了保证RDB文件的完整性，子进程在操作时会先生成一个临时文件，操作完成之后再将临时文件重新命名为xxx.rdb(配置文件配置)，其中forkv操作时会阻塞redis服务。

 COW写时复制：当子线程和主线程都需要操作同一块内存的情况，主线程会先进行复制再进行操作复制。


2. redis.conf配置

在配置文件中配置 ```save m n``` 规则自动触发 m秒内有n次操作就会进行一次备份，执行SHOTDOWN命令时，如果没有开启AOF,也会触发RDB操作。

```sh

# 时间策略  900秒内有一个key的值变化就会触发rdb
save 900 1

# rdb 文件的名称
dbfilename dum.rdb

# rdb文件保存的路径
dir ./

# rdb文件是否进行压缩
rdbcompression yes


```

###### 总结

- 优点

执行效率高，适合大规模数据备份，不会影响主线程工作，备份文件占用空间小。

- 缺点

在执行RDB间隔间的数据可能会丢失，fork操作时会阻塞主线程。




##### AOF


- AOF:记录每次对服务器写的操作，当服务重启时执行这些命令来恢复数据。（默认不开启）


AOF持久化的实现


```sh
命令写入 ->  AOF缓冲  ->  AOF文件


# redis.conf配置

# 开启aof
appendonly yes
# 命名
appendfilename "appendonly.aof"

# 同步策略
appendfsync always  # 执行一次命令 立刻同步到aof文件 消耗大 最安全
appendfsync everysec  # 每一秒同步一次
appendfsync no  # 有操作系统同步
```

先放入AOF缓冲区，不让磁盘IO成为redis性能瓶颈。当redis重启时，会重新加载AOF文件。


- AOF问题

    - AOF文件体积膨胀? 解决： redis提供了AOF文件重写策略，FORK出子线程进行重写
    
    重写触发：

    1. 手动触发，bgrewriteaof命令
    2. 配置文件

    ```sh
    # 当前AOF文件的大小是上次AOF文件的100% 并且体积达到64M 满足两者才会触发
    auto-aof-rewrite-percentage 100
    auto-aof-min-size 64mb

    ```


##### RDB和AOF优先级

优先AOF



##### redis持久化总结

1. RDB和AOF都会FORK子线程，会阻塞主线测，所以应该降低FORK触发频率，
2. 控制redis最大使用内存，防止FORK时间太长
3. RDB和AOF可以同时存在，配合使用



#### redis过期删除策略&内存淘汰策略

设置redis键过期时间

```sh

expire key ttl  # 过期 秒
pexpire key ttl  # 过期 毫秒
expireat key timestamp  # 过期 秒时间戳
pexpireat key timestamp # 过期 毫秒时间戳
persitt key # 移除过期时间
ttl key # 查看过期时间 秒
pttl key # 查看过期时间 毫秒
```

##### redis过期删除策略

1. 惰性删除： 获取的时候判断如果过期了就删除
2. 定期删除： 每隔一段时间随机选出一定数量的键，删除其中过期的键，默认随机选出10个。


##### redis内存淘汰策略

redis.conf设置最大内存

```sh
maxmemory bytes # 通常设定物理内存的3/4
maxmemory-policy volatile-lru  # 设置淘汰策略
```

换成内存淘汰策略中有 ```FIFO```,```LRU``` ,```LFU```三种，redis在使用的有```LRU``` 和```LFU```

```FIFO```: 先进先出。
```LRU``` : 最近最少使用，核心思想，如果最近被使用，那么将来会被用到的概率变高，如果key最近被访问了，则移动到链表头部，redis中随机取出5个数据，选择时间最早的进行淘汰，存在一种情况：热点数据最近很少访问有可能被删除。
```LFU``` : 最不经常使用，过去访问的次数越多，将来被访问的概率也越高。


淘汰策略

1. ```noeviction``` : 默认策略，内存到达最大值，所有写操作都报错。
2. ```volatile-ttl``` : 过期时间越早被淘汰
3. ```allkeys-random``` : 随机
4. ```volatile-random``` : 有过期时间的key随机
5. ```xxx-lru``` : ```LRU```算法
6. ```xxx-lfu ```: ```LFU```算法



#### 性能压测

```redis-benchmark```


#### redis高可用


##### 主从架构

1. 全量复制

```master```将自己的```RDB```文件发送给```slave```,进行数据同步，记录同步期间的其它写入，再发送给```slave```。

2. 增量复制

```master```和```slave```短时间断开，```salve```发送```offset```，```master```根据偏移量发送偏移数据给```slave```，如果时间太长，则需要全量复制。


配置主从配置：

```sh

1、 配置文件 slaveof <masterip><masterport>
2、 命令执行 redis-server --slaveof <masterip><masterport>
3、 客户端命令执行 slaveof <masterip><masterport>

```

主节点挂了之后从节点不会自动升级为节点，这时可以加上redis哨兵实现


##### sentinel

1. 添加配置文件 ```sentinel.conf```

```sh
sentinel monitor <名称> <ip> <port> <得票数>
```

2. 启动 ``` redis-sentinel ./sentinel.conf ```

3. 选举策略 ： 复制进度大的 / ID号小的


问题：

- 仍然只有一个master节点处理写请求，并发请求量大时，服务任然压力大。
- 每个从节点需要保存全量的数据，冗余的数据量大

##### redis cluster

redis clustor 采用多主多从的方式，引入哈希槽（16384）的概念，每个主节点只保存一部分数据，当某个主节点挂了之后从节点自动升级为主节点，

1. 配置文件添加

```sh
cluster-enabled yes
```

2. 启动多个服务

3. 构建cluster

```sh
redis-cli --cluster create <ip:port> <ip:port> <ip:port> --cluster-replicas 1
```

cluster-replicas表示master节点下需要有几个从节点，这样cluster就会将16384个槽位平均分配到每个节点。

4. 扩容时 先添加节点到cluster中，然后重新分配槽位到新的节点

```sh
# 添加节点
redis-cli --cluster add-node <需要添加的节点ip:port> <任意已存在节点 ip:port>

# 查询节点信息
cluster nodes

# 添加重新分片
redis-cli --cluster reshard <任意已存在节点ip:port> --cluster-from <已存在节点ID> --cluster-to <新添加节点ID> --cluster-slots <需要分配槽的个数>
```


