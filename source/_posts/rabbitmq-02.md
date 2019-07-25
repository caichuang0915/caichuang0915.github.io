---
title: RabbitMq - 配置和相关命令
tags:
  - RabbitMq
---

RabbitMq 修改端口/用户权限 相关命令

## RabbitMq 基相关命令

- 配置文件

> RabbitMq配置文件在/etc/rabbitmq下,名为rabbitmq.config / rabbitmq-env.conf,需要自己创建。

	修改端口:
	创建rabbitmq-env.conf文件,添加RABBITMQ_NODE_PORT=8500,重启RabbitMq即可生效.

- 启动和关闭RabbitMq

```bash
rabbitmq-server #会启动Erlang节点和Rabbitmq应用 
rabbitmqctl stop #会关闭Erlang节点和Rabbitmq应用 
rabbitmqctl status  #可以检查消息节点是否正常

#单独关闭RabbitMQ应用
rabbitmqctl stop_app #关闭Rabbitmq应用 
rabbitmqctl start_app #启动Rabbitmq应用
```

<!-- more -->

- 管理虚拟主机

```bash
rabbitmqctl add_vhost tupelo # 创建vhost
rabbitmqctl list_vhosts   # 列出所有的vhost
```

- 用户管理

```bash
rabbitmqctl add_user tupelo 123456 # 创建用户
rabbitmqctl  delete_user  Username # 删除用户
rabbitmqctl  change_password  Username  Newpassword # 修改用户
rabbitmqctl list_users # 查询所有的用户
```

- 用户权限管理(guest特殊用户)

> guest是默认用户，具有默认virtual host "/"上的全部权限，仅能通过localhost访问RabbitMQ包括Plugin，建议删除或更改密码。可通过将配置文件中loopback_users来取消其本地访问的限制：[{rabbit, [{loopback_users, []}]}] 

> 用户仅能对其所能访问的virtual hosts中的资源进行操作。这里的资源指的是virtual hosts中的exchanges、queues等，操作包括对资源进行配置、写、读。配置权限可创建、删除、资源并修改资源的行为，写权限可向资源发送消息，读权限从资源获取消息。比如：  
exchange和queue的declare与delete分别需要：exchange和queue上的配置权限  
> queue的bind与unbind需要：queue写权限，exchange的读权限  
> 发消息(publish)需exchange的写权限  
> 获取或清除(get、consume、purge)消息需queue的读权限  
> 对何种资源具有配置、写、读的权限通过正则表达式来匹配，具体命令如下：  
> rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>  

```bash
#如用户Mark在虚拟主机logHost上的所有权限： 
rabbitmqctl set_permissions –p logHost Mark  '.*'  '.*'  '.*'
# 列出用户的权限
rabbitmqctl  list_user_permissions  User  
# 清空(某一个vhost)用户的权限
rabbitmqctl  clear_permissions  [-p VHostPath]  User
# 查询(某一个vhost)所有用户的权限
rabbitmqctl  list_permissions  [-p  VHostPath]
```


- 用户角色

```bash
# 设置用户角色
rabbitmqctl  set_user_tags  tupelo administrator 
# 设置用户多个角色
rabbitmqctl  set_user_tags  tupelo  monitoring  policymaker
```

- 用户角色分类 (none、management、policymaker、monitoring、administrator)

> none、management、policymaker、monitoring、administrator

	(1) 超级管理员(administrator)
		policymaker和monitoring可以做的任何事加:
		创建和删除virtual hosts
		查看、创建和删除users
		查看创建和删除permissions
		关闭其他用户的connections

	(2) 监控者(monitoring)
		management可以做的任何事加：
		列出所有virtual hosts，包括他们不能登录的virtual hosts
		查看其他用户的connections和channels
		查看节点级别的数据如clustering和memory使用情况
		查看真正的关于所有virtual hosts的全局的统计信息

	(3) 策略制定者(policymaker)
		management可以做的任何事加：
		查看、创建和删除自己的virtual hosts所属的policies和parameters

	(4) 普通管理者(management)
		普通的生产者和消费者加：
		列出自己可以通过AMQP登入的virtual hosts  
		查看自己的virtual hosts中的queues, exchanges 和 bindings
		查看和关闭自己的channels 和 connections
		查看有关自己的virtual hosts的“全局”的统计信息，包含其他用户在这些virtual hosts中的活动。

	(5) 其他(none)
		不能访问 management plugin，通常就是普通的生产者和消费者

- 其他命令

```bash
rabbitmqctl list_queues    # 查看所有的队列
rabbitmqctl list_exchanges # 查看所有的交换机
rabbitmqctl list_bindings  #查看所有的绑定
```










