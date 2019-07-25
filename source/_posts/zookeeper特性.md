---
title: Zookeeper特性
tags:
  - zookeeper
---

#### 目标

- 简单的数据结构：共享的树形结构，类似文件系统，存储于内存；
- 可以构建集群：避免单点故障，3-5台机器就可以组成集群，超过半数正常工作就能对外提供服务；
- 顺序访问：对于每个读请求，zk会分配一个全局唯一的递增编号，利用这个特性可以实现高级协调服务；
- 高性能：基于内存操作，服务于非事务请求，适用于读操作为主的业务场景。3台zk集群能达到13w QPS；

#### 使用场景

- 数据发布订阅
- 负载均衡
- 命名服务
- Master选举
- 集群管理
- 配置管理
- 分布式队列
- 分布式锁

<!-- more -->  

#### 特性

- 会话

	客户端与服务端的一次会话连接，本质是TCP长连接，通过会话可以进行心跳检测和数据传输

- ZK数据模型

	ZooKeeper的视图结构和标准的Unix文件系统类似，其中每个节点称为“数据节点”或ZNode,每个znode可以存储数据，还可以挂载子节点，因此可以称之为“树”。第二点需要注意的是，每一个znode都必须有值，如果没有值，节点是不能创建成功的。

- Zookeeper节点类型

	永久/临时/永久顺序/临时顺序

- Zookeeper节点状态属性

	![节点属性](http://image.tupelo.top/zk%E8%8A%82%E7%82%B9%E5%B1%9E%E6%80%A7.png)

- ACL保障数据的安全

	ACL机制，表示为scheme:id:permissions，第一个字段表示采用哪一种机制，第二个id表示用户，permissions表示相关权限（如只读，读写，管理等）。

#### 命令行

服务端：

```sh
zkServer start/restart/stop/status
```

客户端：

```sh
# 默认连本机
zkCli -server 127.0.0.1:2181

# 查看
ls /
# 查看详情
ls2 /
# 创建节点 -e 临时节点 -s 顺序节点
create -e -s /aa
# 查询/赋值
get/set /aa a 
# 删除
delete /aa
# 递归删除
rmr /aa
```

#### ACL命令常用命令

Zookeeper的ACL(Access Control List)，分为三个维度：scheme、id、permission		
通常表示为：scheme\:id:permissions,第一个字段表示采用哪一种机制，第二个id表示用户，permissions表示相关权限（如只读，读写，管理等）。

- schema:代表授权策略
- id:代表用户
- permission:代表权限

Scheme:

- world：

	默认方式，相当于全世界都能访问

- auth：

	代表已经认证通过的用户(可以通过addauth digest user:pwd 来添加授权用户)

- digest：

	即用户名:密码这种方式认证，这也是业务系统中最常用的

- ip：

	使用Ip地址认证

id 验证模式，不同的scheme，id的值也不一样:

- scheme为auth时 username:password
- scheme为digest时 username:BASE64(SHA1(password))
- scheme为ip时 客户端的ip地址。
- scheme为world时 anyone。

permission:

CREATE、READ、WRITE、DELETE、ADMIN 也就是 增、删、改、查、管理权限，这5种权限简写为crwda(即：每个单词的首字符缩写)

命令：

```sh
# 查看权限
getAcl /aa
# 添加权限
setAcl /aa world:anyone:crwa
# 添加用户 断开连接后 用户也退出 二次执行时前一个用户也退出
addauth digest user1:123456 
# auth与digest的区别就是，前者使用明文密码进行登录，后者使用密文密码进行登录
setAcl /testDir/testAcl auth:user1:123456:crwa
setAcl /testDir/testDigest digest:user1:HYGa7IZRm2PUBFiFFu8xY2pPP/s=:crwa

# 获取密文 直接调用jar包
java -Djava.ext.dirs=/soft/zookeeper-3.4.12/lib -cp /soft/zookeeper-3.4.12/zookeeper-3.4.12.jar org.apache.zookeeper.server.auth.DigestAuthenticationProvider deer:123456
```

setAcl权限的时候由于失误，导致节点无法删除		
解决方式:启用super权限				
使用```DigestAuthenticationProvider.generateDigest("super:admin");``` 获得密码
修改zkServer启动脚本增加```-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs=```		
启动客户端用管理员登陆进行删除```addauth digest super:admin```	

#### ZK四字命令

ZooKeeper 支持某些特定的四字命令字母与其的交互。用来获取 ZooKeeper 服务的当前状态及相关信息。可通过 telnet 或 nc 向 ZooKeeper 提交相应的命令

- echo stat|nc 127.0.0.1 2181 来查看哪个节点被选择作为follower或者leader 
- echo ruok|nc 127.0.0.1 2181 测试是否启动了该Server，若回复imok表示已经启动。 
- echo dump| nc 127.0.0.1 2181 ,列出未经处理的会话和临时节点。 
- echo kill | nc 127.0.0.1 2181 ,关掉server 
- echo conf | nc 127.0.0.1 2181 ,输出相关服务配置的详细信息。 
- echo cons | nc 127.0.0.1 2181 ,列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息 
- echo envi |nc 127.0.0.1 2181 ,输出关于服务环境的详细信息（区别于 conf 命令）。 
- echo reqs | nc 127.0.0.1 2181 ,列出未经处理的请求。 
- echo wchs | nc 127.0.0.1 2181 ,列出服务器 watch 的详细信息。 
- echo wchc | nc 127.0.0.1 2181 ,通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。 
- echo wchp | nc 127.0.0.1 2181 ,通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。

#### ZooKeeper日志可视化

> 直接调用jar包命令,两个非常重要的配置一个是dataDir，存放的快照数据，一个是dataLogDir，存放的是事务日志文件


```sh
# 事务日志
/soft/zookeeper-3.4.12/zookeeper-3.4.12.jar:/soft/zookeeper-3.4.12/lib/slf4j-api-1.7.25.jar org.apache.zookeeper.server.LogFormatter log.1
# 快照日志
/soft/zookeeper-3.4.12/zookeeper-3.4.12.jar:/soft/zookeeper-3.4.12/lib/slf4j-api-1.7.25.jar org.apache.zookeeper.server.SnapshotFormatter log.1
```
