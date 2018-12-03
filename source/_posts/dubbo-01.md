---
title: Dubbo基础 - 01
tags:
  - Dubbo
---

Dubbo基础学习配置

## Dubbo 基本使用和配置

dubbo主要是使不同的服务通过注册中心互相调用，可以解耦。但是依赖于spring,就是一个spring项目和一个python项目无法通过dubbo互相调用，通过consul可以互相调用。

问题：dubbo调用和maven依赖有什么区别(待查证)


### Dubbo配置的四种方式

> XML配置 (使用最多)

> 注解配置

> 配置文件配置

> API配置（研究Dubbo的入口）

<!-- more -->
- 添加dubbo所需要的依赖（zkclient dubbo）两个

```java
<dependency>
	<groupId>com.101tec</groupId>
	<artifactId>zkclient</artifactId>
	<version>0.3</version>
</dependency>

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>dubbo</artifactId>
	<version>2.5.7</version>
	<scope>compile</scope>
	<exclusions>
		<exclusion>
			<artifactId>spring</artifactId>
			<groupId>org.springframework</groupId>
		</exclusion>
	</exclusions>
</dependency>
```

- 新建dubbo的xml配置文件 首先是生产者provider的配置文件（以product为例）

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://code.alibabatech.com/schema/dubbo
             http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息 用于计算依赖关系 -->
    <dubbo:application name="productService"/>
    <!-- 调用协议 -->
    <dubbo:protocol name="dubbo" port="20880"/>
    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>

	<!-- 生成远程服务代理 可以和本地bean一样使用demoService  -->
    <dubbo:service interface="com.tupelo.dubboproduct.service.Product2Service" ref="product2Service" protocol="dubbo"/>
</beans>
	
```

消费者consumer配置文件

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	 http://www.springframework.org/schema/beans/spring-beans.xsd        
	 http://code.alibabatech.com/schema/dubbo        
	 http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息 用于计算依赖关系 -->
    <dubbo:application name="enjoyStore"/>

    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>

    <dubbo:consumer check="false"/>

    <!-- 生成远程服务代理 可以和本地bean一样使用demoService -->
    <dubbo:reference id="userService" interface="com.enjoy.service.UserService"  />
</beans>
```

- 配置文件配置方式 在properties文件中配置相关属性补充xml中没有的属性 优先级最低 (yml文件为例)

```java
dubbo:
  protocol:
    name: dubbo
    port: 20880
  application:
    name: productService
  registry:
    address: zookeeper://localhost:2181
```

- 注解方式（半注解方式 springboot是使用全部注解）使用<dubbo:annotation>去代替<dubbo:reference>和<dubbo:service>标签 同时生产者Service注解使用dubbo的注解 消费者的调用时使用@Reference注解代替@Autowired注解

```java

<dubbo:annotation package="com.cc.controller" />

调用时：
	//@Autowired
	@Reference
    private ProductService productService;

实现类：
	//@org.springframework.stereotype.Service
	@com.alibaba.dubbo.config.annotation.Service
	public class ProductServiceImpl implements ProductService(){...}
```

- API方式（待定）



