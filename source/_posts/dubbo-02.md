---
title: Dubbo基础 - 配置文件
tags:
  - Dubbo
---

Dubbo基础学习配置

## Dubbo 配置文件

- 消费方配置  

> check

```java
 <dubbo:consumer timeout="3000" check="false"/>
```

check = false 防止启动报错 没有提供者会自动生成代理(测试环境使用)  
check = true 没有提供者直接报错(正式环境使用)

<!-- more -->

> cluster 负载策略(常用的两种)

```java
<dubbo:reference id="productService" interface="com.tupelo.service.ProductService" cluster="failover" />
```  
failover 重试其他服务 retries 重试次数  
failfast 直接报错

> loadbalance 负载配置

```java
//消费者
<dubbo:reference id="productService" interface="com.tupelo.service.ProductService"loadbalance="random" />

// 提供者
<dubbo:service interface="com.tupelo.dubboproduct.service.ProductService" ref="productService" protocol="dubbo" weight="20"/>

```  
random   按权重随机（权重 服务方配置 weight）  
roundRobin 轮询  
leastActive 按活跃程度

> 开启缓存

```java
<dubbo:reference id="productService2" interface="com.tupelo.dubboproduct.service.Product2Service">
        <dubbo:method name="getProduct" cache="lru"></dubbo:method>
</dubbo:reference>
```  
random   按权重随机（权重 服务方配置 weight） 
roundRobin 轮询  
leastActive 按活跃程度  
..(自定义缓存)

> 异步请求 async="true" 通常是配合时间通知一起处理 单独处理使用Future类(不提倡)

```java
<dubbo:reference id="productService2" interface="com.tupelo.dubboproduct.service.Product2Service" async="true">
    <dubbo:method name="getProduct" cache="lru" onreturn="returnCallback" onthrow="errorCallback"></dubbo:method>
</dubbo:reference>
```  

> 事件通知 回调方法一个参数为返回参数 其它参数为入参

```java
<dubbo:reference id="productService2" interface="com.tupelo.dubboproduct.service.Product2Service">
    <dubbo:method name="getProduct" cache="lru" onreturn="returnCallback" onthrow="errorCallback"></dubbo:method>
</dubbo:reference>
```  
onreturn 正常返回回调  
onthrow 方法异常回调

> 回声测试 检查服务是否已经就绪

```java 
public HashMap test(HttpServletRequest request, HttpServletResponse response) {
        String[] serviceIds = new String[]{"productService","userService","orderService","payService"};
        HashMap<String,String> retMap = new HashMap<>();

        Object ret = null;
        for (String id:serviceIds){
            try {
                EchoService echoService = (EchoService)ctx.getBean(id);
                ret = echoService.$echo("ok");
                retMap.put(id,ret.toString());
            } catch (Exception e) {
                retMap.put(id,"not ready");
            }
        }

        return retMap;

    }
```

> 泛化调用 当A项目没有得到B项目的接口描述 但是还是想调用B项目的接口

1 配置dubbo引入，设置generic = true
```java
<dubbo:reference id="otherService" interface="com.enjoy.service.OtherService" generic="true" />
```
2 从IOC容器中取出代理对象，转为泛型接口对象 通过$invoke方法调用目标方法（传入方法名/参数类型/参数值）

```java
public String other(HttpServletRequest request, HttpServletResponse response) {
        GenericService genericService = (GenericService)ctx.getBean("otherService");
        Object ret = genericService.$invoke("getDetail",new String[]{"java.lang.String"},new Object[]{"name"});
        return ret.toString();
    }
```