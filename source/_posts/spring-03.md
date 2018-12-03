---
title: Spring学习 - 03
tags:
  - Java
  - Spring
---

spring基础学习

### bean的生命周期

bean的生命周期:指   bean创建---初始化---销毁  的过程

bean的生命周期是由容器进行管理的.我们可以自定义 bean初始化和销毁方法: 容器在bean进行到当前生命周期的时候, 来调用自定义的初始化和销毁方法

#### 自定义 bean初始化和销毁方法


- 指定初始化和销毁方法 init-method和destory-mothod

```java
	@Bean(initMethod="init",destoryMothod="destroy")
	public Person lison(){
		return new Person();
	}
```

容器关闭时，只有单实例Bean会调用distory方法，多实例容器只负责初始化,但不会管理bean, 容器关闭不会调用销毁方法。

<!-- more -->

- 让Bean实现 InitializingBean 和 DisposableBean接口

afterPropertiesSet()方法:当beanFactory创建好对象,且把bean所有属性设置好之后,会调这个方法,相当于初始化方法
destory()方法,当bean销毁时,会把单实例bean进行销毁

```java
	@Component
	public class People implements InitializingBean,DisposableBean {
	    @Override
	    public void destroy() throws Exception {
	        System.out.println("销毁");
	    }

	    @Override
	    public void afterPropertiesSet() throws Exception {
	        System.out.println("初始化");
	    }
	}
```

- JSR250规则定义的注解 @PostConstruct和@PreDestroy

@PostConstruct: 在Bean创建完成,且属于赋值完成后进行初始化,属于JDK规范的注解
@PreDestroy: 在bean将被移除之前进行通知, 在容器销毁之前进行清理工作

```java
	@Component
	public class People {
	    @PostConstruct
	    public void init(){
	        System.out.println("初始化");
	    }
	    @PreDestroy
	    public void distory(){
	        System.out.println("销毁");
	    }
	}
```

- 后置处理器BeanPostProcessor,在bean初始化之前调用进行拦截

	- postProcessBeforeInitialization():在初始化之前进行后置处理工作(在init-method之前),    
	- postProcessAfterInitialization():在初始化之后进行后置处理工作

```java
	@Component
	public class People implements BeanPostProcessor {
	    @Nullable
	    @Override
	    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println("初始化");
	        return null;
	    }

	    @Nullable
	    @Override
	    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	        System.out.println("销毁");
	        return null;
	    }
	}
```

