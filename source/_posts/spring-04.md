---
title: Spring学习04 - BeanPostProcessor源码分析
tags:
  - Java
  - Spring
---

spring基础学习

### 容器启动及BeanPostProcessor源码分析

BeanPostProcessor原理:
可从容器类跟进顺序为:
```
AnnotationConfigApplicationContext-->refresh()-->
finishBeanFactoryInitialization(beanFactory)--->
beanFactory.preInstantiateSingletons()-->
760行getBean(beanName)--->
199行doGetBean(name, null, null, false)-->
317行createBean(beanName, mbd, args)-->
501行doCreateBean(beanName, mbdToUse, args)-->
541行createBeanInstance(beanName, mbd, args)(完成bean创建)-->
578行populateBean(beanName, mbd, instanceWrapper)(属性赋值)-->
579行initializeBean(beanName, exposedObject, mbd)(Bean初始化)->
1069行到1710行,后置处理器完成对init方法的前后处理.
```

initializeBean方法 对方法前后做处理
```java
{
  applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  invokeInitMethods(beanName, wrappedBean, mbd) //执行自定义初始化
  applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName)
}
```








