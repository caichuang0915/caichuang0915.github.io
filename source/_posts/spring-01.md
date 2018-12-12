---
title: Spring学习01 - 工程从XML配置到注解
tags:
  - Java
  - Spring
---

spring基础学习

## Spring 基础及组件使用01


### 工程从XML配置到注解

- 创建配置类 加上@Configuration注解 加上@ComponentScan注解 扫描指定的包 加载到IOC容器

```java
	@Configuration
	@ComponentScan(value = "com.tupelo",useDefaultFilters = true,excludeFilters = {
	        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
	})
	@EnableAspectJAutoProxy
	public class MainConfig {

	    @Bean
	    public Person person(){
	        return new Person("jack",20);
	    }

	}
```
<!-- more -->

- 启动时加载配置类

```java
	public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
    }
```

### scop扫描规则

- prototype  多实例 IOC初始化的时候不会创建类 使用的时候才创建(懒加载也是 懒加载是针对单实例bean)
- singleton  单实例
- request    一次请求创建一个类
- session    同一个session创建一个类

```java
	@Lazy
 	// @Scope("prototype")
	@Bean
    public Person person(){
        return new Person("jack",20);
    }

```



