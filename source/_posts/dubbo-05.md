---
title: Dubbo - 总结
tags:
  - Dubbo
---

Dubbo 总结

> 参考官方文档：http://dubbo.apache.org/zh-cn/docs/user/references/xml/dubbo-service.html

<!-- more -->

## Dubbo 总结

#### dubbo初始化流程

- 入口
    - spring标签DubboNameSpaceHandler
    - referencebean + servicebean 
    - 继承了spring initalizingBean接口
    - afterPropertiesSet
    - protocol.refer(url) + protocol.export(url)+(registryURL)

- 消费端
    - RegistryProtocol.refer
    - zkRegistry.subscribe
    - 触发监听器回调
    - registryDictory 
    - protocol.refer 
    - dubboProtocol.refer + interface
    - 消费方interface代理对象

- 服务端
    - RegistryProtocol.export 
    - protocol.export 
    - 创建中转对象
    - zkRegistry.subscribe注册


#### spi

jdk的spi：把实现类,装入一个list中
dubbo的spi：把实现类,装入一个map中(配置文件指定一key),key指代实现类

#### RegistryProtocol

dubbo偷懒，借用protocol流程模块，把注册模块，伪装成protocol协议(注册协议),注册协议跟其它不一样，任何协议dubbo/rmi/rest,要生效,都必须注册,因此RegistryProtocol是最优先被调用，然后再转给真实的协议

#### 服务治理

- dubbo只是服务治理，springcloud包含所有全套微服务功能
- dubbo把rpc做了成透明化的调度,减少出错机率(spring cloud易出错)
- springcloud的远程调用，没能完全透明化。透明化调度，意思你使用它，像本地服务一样用。
- dubbo控制台，可以关闭某个服务，不对外开放（开发环境常用）
