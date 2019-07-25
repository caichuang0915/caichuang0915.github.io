---
title: Zookeeper应用(服务注册与发现/分布式锁/Master选举/配置中心)
tags:
  - zookeeper
---

#### 服务注册与发现

注册：

> 服务启动时 在某一个节点下新建一个临时顺序节点将ip+端口保存

```java

public static  void register(String address,int port) {
        try {
            ZooKeeper zooKeeper = new ZooKeeper("localhost:2181",5000,(watchedEvent)->{});
            logger.warn("连接zk");
            Stat exists = zooKeeper.exists(BASE_SERVICES + SERVICE_NAME, false);
            logger.warn("判断是否存在" + exists);
            if(exists==null) {
                logger.warn("创建路径" );
                zooKeeper.create(BASE_SERVICES,"".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
                zooKeeper.create(BASE_SERVICES + SERVICE_NAME,"".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

            }
            String server_path = address+":"+port;
            logger.warn("server_path:" + server_path);
            zooKeeper.create(BASE_SERVICES + SERVICE_NAME+"/child",server_path.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
            logger.warn("保存成功" );
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

发现：

> 监听某一个节点，当子节点发生变化时更新ip列表，请求时从列表中获取ip+端口

```java
private void updateServiceList() {
       try{

           logger.warn("ip 改变 开始更新ip地址");
           List<String> children = zooKeeper.getChildren(BASE_SERVICES  + SERVICE_NAME, true);
           List<String> newServerList = new ArrayList<String>();
           for(String subNode:children) {
               byte[] data = zooKeeper.getData(BASE_SERVICES  + SERVICE_NAME + "/" + subNode, false, null);
               String host = new String(data, "utf-8");
               System.out.println("host:"+host);
               newServerList.add(host);
           }
           LoadBalance.SERVICE_LIST = newServerList;
       }catch (Exception e) {
           e.printStackTrace();
       }
    }
```
<!-- more --> 

#### 分布式锁

- 基于同名节点的分布式锁

思想：都创建同一个临时节点，成功获得锁，失败等待

![分布式锁](http://image.tupelo.top/%E4%B8%B4%E6%97%B6.png)

- 高性能分布式锁

思想：创建临时顺序节点 若为最小节点，获得锁，否则监听上一个节点是否被删除

![分布式锁](http://image.tupelo.top/%E6%80%A7%E8%83%BD.png)
 

#### 自动选举

思想：监听某一个节点，当节点被删除时，创建临时节点写入自己的信息，成功则自己为master节点，否则获取该节点信息为master信息，监听该节点。


#### 数据发布与订阅（配置中心）

思想：直接监听某一个节点，当节点内容有修改是，直接修改配置。





