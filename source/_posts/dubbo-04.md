---
title: Dubbo - Spi机制
tags:
  - Dubbo
---

Dubbo Spi机制

## Dubbo Spi机制

本质是解决同一个接口，有多种实现时，使用者如何能够方便选择实现的问题


<!-- more -->


#### Spi机制(策略模式)


- jdk的Spi机制

![jdk的Spi机制](http://image.tupelo.top/jdkspi.png)

jdk中，选择SpiService的实现，方法是在jar中放置一个META-INF/services目录，目录中存放一个文本文件(文件名----是SpiService接口的全路径名)，文本中列入你选择的实现类(一行放一个------是实现类的全路径名),有了上述配置，在java程序中，使用ServiceLoader.load(SpiService.class)，
即可将配置中选择的实现类，实例化并放入一个集合中

```java
//配置文件
com.enjoy.spi.service.RmiSpiServiceImpl
com.enjoy.spi.service.RestSpiServiceImpl

//加载
public static void main(String[] args) throws IOException {
        //加载接口
        ServiceLoader<SpiService> spiLoader = ServiceLoader.load(SpiService.class);
        Iterator<SpiService> iteratorSpi = spiLoader.iterator();
        while (iteratorSpi.hasNext()) {
            SpiService spiService = iteratorSpi.next();
            spiService.sayHello();
        }
}
```

- Dubbo的Spi机制

![spi机制](http://image.tupelo.top/spi%E6%9C%BA%E5%88%B6.png)

```java
<dubbo:reference  id="productService" cluster="failover" interface="com.enjoy.service.ProductService" />
<dubbo:reference  id="productService1" cluster="failsafe" interface="com.enjoy.service.ProductService" />
<dubbo:reference  id="productService2" cluster="failfast" interface="com.enjoy.service.ProductService" />
```

与jdk相比,dubbo将选择权下放到了配置文件中(你配置谁，它使为你实例化谁)dubbo的目标,以cluster为例,failsafe/failover/failfast都是cluster的一种实现,现在我    们可以在标签配置时,方便地进行选择.凡是dubbo中,接口上有 @SPI 标注的，都表明此接口支持扩展.


- 自定义扩展(LoadBalance为例)

引入maven支持
```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.7</version>
    <scope>provided</scope>
</dependency>
```

新建一个类 定义使用第一个服务的负载规则
```java
public class FirstLoadBalance implements LoadBalance {//filter
    /**
     * @param invokers 所有provider的实现
     * @param url
     * @param invocation
     * @param <T>
     * @return
     * @throws RpcException
     */
    @Override
    public <T> Invoker<T> select(List<Invoker<T>> invokers, URL url, Invocation invocation)
            throws RpcException {
        System.out.println("启动第一个");
        //固定使用第一个
        return invokers.get(0);
    }
}
```

添加配置文件 路径为 resources/META-INF/dubbo,文件名和接口(LoadBalance)全路径名一样,com.alibaba.dubbo.rpc.cluster.LoadBalance,以 key-value 方式添加内容

```java
first=com.enjoy.loadbalance.FirstLoadBalance
```

在消费方dubbo配置文件中就可以使用自定义规则了

```java
<dubbo:reference  id="userService" interface="com.enjoy.service.UserService" loadbalance="first" />

```


#### Spi实现原理---核心ExtensionLoader

protocol是个代理类，不是真实的协议实现

```java
private static final Protocol refprotocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

    ExtensionLoader的加载步骤
    1、getExtensionLoader(Protocol.class)为protocol接口生成一个加载器
    2、getAdaptiveExtension()，使用加载器生成一个代理对象---- protocol接口对象
    3、代理对象执行时，根据参数（扩展名extName）选择实际对象
    4、每个接口扩展点,对应一个ExtensionLoader加载器，如：
       protocol      ExtensionLoader实例<protocol>
       filter        ExtensionLoader实例<filter>
       loadbalance   ExtensionLoader实例<loadbalance>

代理类的创建，是通过动态代码，生成一个类源码，然后经过编译得到代理类的class(Demo)(代码hexo报错,原因待查,直接上图片)
![demo](http://image.tupelo.top/demo.png)

- Dubbo的ExtensionLoader执行逻辑

![Dubbo的ExtensionLoader执行逻辑](http://image.tupelo.top/extension%E6%89%A7%E8%A1%8C%E9%80%BB%E8%BE%91-2.png)

    a、dubbo启动加载实现类时，以   key-实例 方式map缓存各个实现类
    b、实际调用时，通过key --取实现需要那个实现
    c、调用的发生，由生成的代理对象的来发起，最终是从URL总线中，找出extName值，extName做为别说，在缓存map中取出正确的实现实现类


- Dubbo的扩展框架

![Dubbo的扩展框架](http://image.tupelo.top/loader%E6%9C%BA%E5%88%B6.png)
- Dubbo的发布订阅
    - 主要是RegistryProtocol,在zk中发布订阅.
    - 当有新的消费节点或者服务节点进入时,触发notify,刷新所有的url信息.
    - zk中的信息,在dubbo代码中,一样要缓存一份(查询会消耗时间),用于容错/负载等动作,此缓存信息的如何与zk中信息保持一致性,依赖发布订阅机制(监听器的及时回调).
    - dubbo注册中心选择很多(数据库功能+发布订阅功能),之所以推荐使用zk,是因为zk在分布式环境中,搭建集群(数据冗余)方便,抗灾性强.


