---
title: 总结-MQ
tags:
  - 总结
---



## kafka

分为生产者、broker、消费者
topic: 虚拟的
分区: 分区，实际真正存放消息的地方，多个分区中有一个主分区，主分区负责消息读取，副本负责消息同步
副本: 一个分区有多个副本
AR: 所有副本集合
ISR: 同步进度相差不大分副本,参与leader选举
OSR: AR - ISR

kafka消费者偏移量的存储

消费者的offset会更新到一个kafka自带的topic consumer_offsets 下面   

kafka的消息丢失

kafka的高可用controller选举

controller实际是在zk中注册了一个临时节点，当controller挂了之后，kafka中集群监听器就会通知其它的broker到zk中创建controler的临时节点，成功了就是新的controller。

kafka文件存储

每个分区都会在所在broker上创建文件夹，一个分区文件又分为多段，每段两个文件，一个日志文件，一个索引文件，日志文件末尾以偏移量命名，查找消息先根据文件名称定位段，然后根据索引文件定位所在位置。


kafka文件删除

时间或者大小

kafka零拷贝

kafka分区一致性



kafka rebalance

kafka数据可靠性

配置响应ACKS,0-无需等待落盘，1-需要等待leader落盘，-1/ALL-所有的节点都落盘

不同的消息语义

至少一次: ACKS = =1
至多一次: ACKS = 0
精准一次: 


## RocketMq

严格的顺序性 （kafka非严格）
拉模型
堆积能力强
满足至少消费一次的语义

支持事务消息 保证最终一致性
18个等级的小时延时
可以指定次数和时间的重试
brocker



## RabbitMq

推模型
