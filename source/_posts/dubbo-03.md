---
title: Dubbo - rpc调用过程
tags:
  - Dubbo
---

Dubbo rpc调用过程

## Dubbo rpc调用过程

- Dubbo的RPC服务暴露和引入

![服务暴露和引入](http://image.tupelo.top/rpc%E6%9C%8D%E5%8A%A1%E6%9A%B4%E9%9C%B2.png)

<!-- more -->

    - 服务暴露：建立一个中转对象（它能调到Impl），来接收网络请求，返回结果到网络
    - 服务引入：找寻到对中转对象，创建一个代理，代理向中转对象传请求参数，等待返回值
    - URL总线：中转对象---URL信息，一一对应，通过URL来找寻中转对象 
    - URL包含完整rpc信息： rmi://192.168.56.1:20881/com.enjoy.service.ProductService?anyhost=true&application=storeServer&dubbo=2.5.7&generic=false&interface=com.enjoy.service.ProductService&methods=modify,getDetail,status&pid=2476&side=provider&timestamp=1542267315993

- Dubbo的消费与服务端连接图示

![连接图示](http://image.tupelo.top/%E6%B6%88%E8%B4%B9%E4%B8%8E%E6%9C%8D%E5%8A%A1%E7%AB%AF%E8%BF%9E%E6%8E%A5.png)

- Dubbo的启动流程

![启动流程](http://image.tupelo.top/dubbo%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png)