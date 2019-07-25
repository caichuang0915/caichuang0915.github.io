---
title: RabbitMq - 集群
tags:
  - RabbitMq
---

RabbitMq集群

## RabbitMq 集群(局域网/网络速度快)

### 目标

实现某一台机器挂了 程序还能运行 并不是为了保证消息万无一失  
> 交换器全局复制 队列只有元数据复制 

#### 相关命令

- rabbitmqctl join_culster [rabbit@node] 加入集群  
- rabbitmqctl status   集群状态  
- rabbitmqctl reset    

<!-- more -->
#### 加入集群

- /etc/host 各节点一直
- /var/lib/rabbitmq/.erlang.cookie各节点一致(修改时可能要改权限777,修改完记得把权限改400回来,否则会报错)
- 启动各节点mq rabbitmq-server
- 要加入节点 先停止 rabbitmqctl stop_app
- 情况集群情况 rabbitmqctl reset
- 加入 rabbitmqctl join_cluster rabbit@node1(重命名方法:修改/etc/rabbitmq/rabbitmq-env.conf,加入 NODENAME=rabbit2@node2),默认是磁盘节点,单参数 --ram 则是内存节点(rabbitmqctl join_cluster rabbit@node1 --ram)
- 启动 rabbitmqctl start_app
- rabbitmqctl status 查看状况

#### 离开集群

- 停止 rabbitmqctl stop_app
- 清空 rabbitmqctl reset
- 启动 rabbitmqctl start_app

#### 镜像队列

> 在定义队列加入相关参数

```java
Map<String, Object> map = new HashMap<String, Object>();
// 是否是在所有节点添加镜像 all是所有
map.put("x-ha-policy","nodes");
// 在指定节点添加镜像
map.put("x-ha-nodes","[rabbit@node1]");
channel.queueDeclare(queueName,false,false,false,map);
```
###### 命令行添加镜像策略

> rabbitmqctl set policy [-p vhost] name pattern Definition 例:queue开头的队列进行镜像处理,节点数1个 

Definition包含三个参数:ha-mode、ha-params、ha-sync-mode  
ha-mode[all、nodes、exactly] //exactly是指个数  
ha-sync-mode[automatic、manual] // 确认方式(自动、手动)  


```
rabbitmqctl set ha_queue "^queue" '{"hamode":"exactly","ha-params":1,"ha-sync-mode":"automatic"}'
```

#### 第三方插件

> 待学习(HAProxy) 类似Nginx







