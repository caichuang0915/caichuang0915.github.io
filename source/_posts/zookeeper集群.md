---
title: Zookeeper集群
tags:
  - zookeeper
---


#### Zookeeper集群

- 顺序一致性
	客户端的更新顺序与它们被发送的顺序相一致。
- 原子性
	更新操作要么成功要么失败，没有第三种结果。
- 单一视图
	无论客户端连接到哪一个服务器，客户端将看到相同的 ZooKeeper 视图。
- 可靠性
	一旦一个更新操作被应用，那么在客户端再次更新它之前，它的值将不会改变。
- 实时性
	连接上一个服务端数据修改，所以其他的服务端都会实时的跟新，不算完全的实时，有一点延时的
- 角色轮换避免单点故障
	当leader出现问题的时候，会选举从follower中选举一个新的leader



#### 集群角色

- Leader  集群工作机制中的核心
	- 事务请求的唯一调度和处理者，保证集群事务处理的顺序性
	- 集群内部个服务器的调度者(管理follower,数据同步)
- Follower  集群工作机制中的跟随者
	- 处理非事务请求，转发事务请求给Leader
	- 参与事务请求proposal投票
参与leader选举投票
- Observer 观察者  
	- 3.30以上版本提供，和follower功能相同，但不参与任何形式投票
	- 处理非事务请求，转发事务请求给Leader
	- 提高集群非事务处理能力

<!-- more -->  

#### 集群配置

配置文件后添加(zoo.cfg)：

```sh
server.0=192.168.212.155:2888:3888
server.1=192.168.212.156:2888:3888
server.2=192.168.212.157:2888:3888
```
	

设置data目录(自定义目录)(zoo.cfg)：

```sh
dataDir=/usr/local/zookeeper/data
```

服务器配置：


```sh
#在/usr/local/zookeeper/data路径创建文件myid并填写内容为0 于server.0对应
```

启动多台zk 单台是无法启动的。

#### Java客户端连接集群

逗号连接

```java
 ZkClient zkClient = new ZkClient("host1,host2,host3");
```





