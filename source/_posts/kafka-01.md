---
title: Kafka学习 - 02
tags:
  - Kafka
  - Zookeeper
---

kafka基础学习-在centos7服务器上搭建kafka

### 安装zookeeper

- 下载zookeeper安装包 [https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/)

- 解压到指定目录 
```bash
tar -zxvf zookeeper-3.4.10.tar.gz -C /usr/local/zookeeper
```

- 修改配置文件
```bash
cd /usr/local/zookeeper/zookeeper-3.4.10/conf/
cp zoo_sample.cfg zoo.cfg
```

<!-- more -->

- 修改zoo.cfg一些基本设置 数据目录 日志目录等

- 配置环境变量(未生效 原因待定)
```bash
vim /etc/profile

export ZOOKEEPER_HOME=/usr/local/zookeeper/zookeeper-3.4.10/
export PATH=$ZOOKEEPER_HOME/bin:$PATH

source /et/profile
```

- 启动zookeeper
```bash
./zkServer.sh start/status/stop/restart
```


### 安装kafka

- 下载kafka安装包 [http://kafka.apache.org/downloads.html](http://kafka.apache.org/downloads.html)


- 解压到指定目录 
```bash
tar -zxvf kafka_2.12-2.0.0.tgz -C /usr/local/kafka/
```

- 配置环境变量

```bash
vim /etc/profile

export KAFKA_HOME=/usr/local/kafka/kafka_2.12-2.0.0
export PATH=$PATH:$KAFKA_HOME/bin

source /et/profile
```


- 验证kafka功能
    - 启动zookeeper
    ```bash
	./zookeeper-server-start.sh -daemon ../config/zookeeper.properties
    ```

    - 启动kafka
    ```bash
	./kafka-server-start.sh ../config/server.properties
    ```

    - 我的服务器为1核1G，kafka启动报错,内存不足Cannot allocate memory
    ```bash
    Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 1073741824, 0) failed; error='Cannot allocate memory' (errno=12)
	#
	# There is insufficient memory for the Java Runtime Environment to continue.
	# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
	# An error report file with more information is saved as:
	# /usr/local/kafka/kafka_2.12-2.0.0/bin/hs_err_pid6279.log
    ```

    - 查看启动脚本 [kafka-server-start.sh] 发现默认配置为：
    ```bash
    if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    	export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
	fi
    ```
    修改配置为 [-Xmx256M -Xms128M] 重启kafka成功

    - 测试创建一个topic
    ```bash
	./kafka-topics.sh --create --zookeeper localhost:2181 --partitions 1 --replication-factor 1 --topic test
    ```

    - 查看topic列表
    ```bash
	./kafka-topics.sh --list --zookeeper localhost:2181
    ```

    - 生产者发送消息
    ```bash
	./kafka-console-producer.sh --broker-list localhost:9092 --topic test
	>haha
    ```

    - 消费者接受消息
    ```bash
    ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic kettle_test --from-beginning
	haha
    ```

    - 集群配置(未配置)...

