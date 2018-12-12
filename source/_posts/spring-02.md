---
title: Spring学习02 - 常用注解
tags:
  - Java
  - Spring
---

spring基础学习

## Spring 基础及组件使用02


### @Conditional条件注册Bean
```java
		package com.enjoy.cap5.config;

		import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
		import org.springframework.context.annotation.Condition;
		import org.springframework.context.annotation.ConditionContext;
		import org.springframework.core.env.Environment;
		import org.springframework.core.type.AnnotatedTypeMetadata;

		public class LinCondition implements Condition{
			/*
			*ConditionContext: 判断条件可以使用的上下文(环境)
			*AnnotatedTypeMetadata: 注解的信息
			*
			*/
			@Override
			public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
				// 业务逻辑
				//能获取到IOC容器正在使用的beanFactory
				ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
				//获取当前环境变量(包括我们操作系统是WIN还是LINUX??)
				Environment environment = context.getEnvironment();
				String os_name = environment.getProperty("os.name");
				if(os_name.contains("linux")){
					return true;
				}
				return false;
			}

		}
```

定义Bean时加上注解
```java
	@Conditional(WinCondition.class)
	@Bean("lison")
	public Person lison(){
		return new Person();
	}
```



### BeanFactory 和 FactoryBean 的区别


- FactoryBean ： 把我们的JAVA实例Bean通过FactoryBean注入到容器中
- BeanFactory ： 从容器中获取我们实例化的Bean

<!-- more -->


### 向IOC容器中注册Bean实例的方法

- @Bean注解: [导入第三方的类或包的组件],比如Person为第三方的类, 需要在我们的IOC容器中使用
- 包扫描+组件的标注注解(@ComponentScan:  @Controller, @Service  @Reponsitory  @ Componet),一般是针对 我们自己写的类,使用这个
- @Import:[快速给容器导入一个组件]
    * @Import(要导入到容器中的组件):容器会自动注册这个组件,bean 的 id为全类名
    ```java
	@Import(value = { Dog.class,Cat.class})
    ```
    * 实现ImportSelector接口:返回需要导入到容器的组件的全类名数组 
    ```java
	import org.springframework.context.annotation.ImportSelector;
	import org.springframework.core.type.AnnotationMetadata;

	public class CaiImportSelector implements ImportSelector{
		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata){
			//返回全类名的bean
			return new String[]{"com.enjoy.cap6.bean.Fish","com.enjoy.cap6.bean.Tiger"};
		}
	}
    ```
    * 实现ImportBeanDefinitionRegistrar接口:可以手动添加组件到IOC容器
    ```java

	public class CaiImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

		/*
		*AnnotationMetadata:当前类的注解信息
		*BeanDefinitionRegistry:BeanDefinition注册类
		*    把所有需要添加到容器中的bean加入;
		*    @Scope
		*/
		@Override
		public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
			boolean bean1 = registry.containsBeanDefinition("com.enjoy.cap6.bean.Dog");
			boolean bean2 = registry.containsBeanDefinition("com.enjoy.cap6.bean.Cat");
			//如果Dog和Cat同时存在于我们IOC容器中,那么创建Pig类, 加入到容器
			//对于我们要注册的bean, 给bean进行封装,
			if(bean1 && bean2){
				RootBeanDefinition beanDefinition = new RootBeanDefinition(Pig.class);
				registry.registerBeanDefinition("pig", beanDefinition);
			}
		}

	}

    ```

- 使用Spring提供的FactoryBean(工厂bean)进行注册 实现FactoryBean接口

```java
import org.springframework.beans.factory.FactoryBean;

// 注入Monkey类
public class CaiFactoryBean implements FactoryBean<Monkey>{

	@Override
	public Monkey getObject() throws Exception {
		return new Monkey();
	}

	@Override
	public Class<?> getObjectType() {
		return Monkey.class;
	}
	
	@Override
	public boolean isSingleton() {
		return true;
	}
}
```

