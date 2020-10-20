---
title: Spring设计模式
tags:
  - Java
  - Spring
---

Spring设计模式

### Spring设计模式


##### 简单工厂模式

BeanFactory。Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。


实现原理：
bean容器的启动阶段：

读取bean的xml配置文件,将bean元素分别转换成一个BeanDefinition对象。
然后通过BeanDefinitionRegistry将这些bean注册到beanFactory中，保存在它的一个ConcurrentHashMap中。
将BeanDefinition注册到了beanFactory之后，在这里Spring为我们提供了一个扩展的切口，允许我们通过实现接口BeanFactoryPostProcessor 在此处来插入我们定义的代码。


简单工厂模式：

- 创建抽象父类
- 子类继承父类实现抽象方法
- 工厂类根据条件产生不同的子类

缺点：
违背了开闭原则,工厂类可能容易成为上帝类。

<!-- more -->


代码：

```java
public abstract class DaBen{ }

public class DaBenC extends DaBen{
    public DaBenC() {
        System.out.println("DaBenC created....");
    }
}

public class DaBenE extends DaBen{
    public DaBenE() {
        System.out.println("DaBenE created....");
    }
}

public class Factory{
    public DaBen createDaBen(String  type){
        if("C".equals(type)){
            return new DaBenC();
        }
        if("E".equals(type)){
            return new DaBenE();
        }
        return null;
    }
}

public static void main(String[] args) {
    Factory factory = new Factory();
    factory.createDaBen("C");
    factory.createDaBen("E");
}
```


##### 工厂方法模式

FactoryBean接口。

实现了FactoryBean接口的bean是一类叫做factory的bean。其特点是，spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个bean.getOjbect()方法的返回值。


工厂方法模式 代码示例：
```java
public class DaBen{ }
public class DaBenC extends DaBen{
    public DaBenC() {
        System.out.println("DaBenC created....");
    }
}
public class DaBenE extends DaBen{
    public DaBenE() {
        System.out.println("DaBenE created....");
    }
}
public interface MainFactory{
    DaBen createDaBen();
}
public class DaBenCCreate implements MainFactory{
    @Override
    public DaBen createDaBen() {
        return new DaBenC();
    }
}
public class DaBenECreate implements MainFactory{
    @Override
    public DaBen createDaBen() {
        return new DaBenE();
    }
}
public static void main(String[] args) {
    MainFactory factoryC = new DaBenCCreate();
    factoryC.createDaBen();
    MainFactory factoryE = new DaBenECreate();
    factoryE.createDaBen();
}
```

扩展 抽象工厂模式：
抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，无须指定它们具体的类。


##### 单例模式

Spring依赖注入Bean实例默认是单例的。Spring的依赖注入（包括lazy-init方式）都是发生在AbstractBeanFactory的getBean里。getBean的doGetBean方法调用getSingleton进行bean的创建。使用了 双重判断加锁 的单例模式。

```java
public Object getSingleton(String beanName){
    //参数true设置标识允许早期依赖
    return getSingleton(beanName,true);
}
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //检查缓存中是否存在实例
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        //如果为空，则锁定全局变量并进行处理。
        synchronized (this.singletonObjects) {
            //如果此bean正在加载，则不处理
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {  
                //当某些方法需要提前初始化的时候则会调用addSingleFactory 方法将对应的ObjectFactory初始化策略存储在singletonFactories
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    //调用预先设定的getObject方法
                    singletonObject = singletonFactory.getObject();
                    //记录在缓存中，earlysingletonObjects和singletonFactories互斥
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```




##### 适配器模式

AOP中的AdvisorAdapter是一个适配器接口，它定义了自己支持的Advice类型，并且能把一个Advisor适配成MethodInterceptor（这也是AOP联盟定义的借口）

```java
public interface AdvisorAdapter {
	// 将一个Advisor适配成MethodInterceptor
	MethodInterceptor getInterceptor(Advisor advisor);
	// 判断此适配器是否支持特定的Advice
	boolean supportsAdvice(Advice advice);
}
```




##### 代理模式


bean初始化的时候的init方法
aopBean初始化的时候直接创建代理类返回

代码示例：

```java
// 接口
public interface Hello {
    public void sayHello();
}
// 实现
public class HelloImpl implements Hello {
    @Override
    public void sayHello(){
        System.out.println("hello........");
    };
}
// JDK代理
public class JdkProxy implements InvocationHandler  {
    private Object object;
    public JdkProxy(Object object) {
        this.object = object;
    }
    public Object newProxy() {
        return Proxy.newProxyInstance(object.getClass().getClassLoader(),
                object.getClass().getInterfaces(), this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("before...........");
        method.invoke(object,args);
        System.out.println("after...........");
        return method;
    }
}
// Cglib代理
public class CglibProxy implements MethodInterceptor {
    private Object object;
    public CglibProxy(Object object) {
        this.object = object;
    }
    public Object createProxyObject() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(object.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before ................");
        method.invoke(object,objects);
        System.out.println("after ................");
        return null;
    }
}
// 调用
@Test
public void testJdkProxy() {
    Hello helloImpl = new HelloImpl();
    JdkProxy interfaceProxy = new JdkProxy(helloImpl);
    Hello hello = (Hello) interfaceProxy.newProxy();
    hello.sayHello();
}

@Test
public void testCglibProxy() {
    Hello helloImpl = new HelloImpl();
    CglibProxy interfaceProxy = new CglibProxy(helloImpl);
    Hello hello = (Hello) interfaceProxy.createProxyObject();
    hello.sayHello();
}
```


##### 模板方法模式

spring jdbcTemplate使用了模板方法模式


模板方法模式 

优点
1、封装不变部分，扩展可变部分。
2、提取公共代码，便于维护。
3、行为由父类控制，子类实现。
缺点
每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

代码：

```java
public abstract class OneDay {
    public abstract void getUp();
    public abstract void sleep();
    public void one(){
        getUp();
        sleep();
    }
}

public class XiaoMingDay extends OneDay{
    @Override
    public void getUp() {
        System.out.println("xiaoming getup ...");
    }
    @Override
    public void sleep() {
        System.out.println("xiaoming sleep ...");
    }
}



public class XiaoWangDay extends OneDay{
    @Override
    public void getUp() {
        System.out.println("xiaoWang getup ...");
    }
    @Override
    public void sleep() {
        System.out.println("xiaoWang sleep ...");
    }
}
public static void main(String[] args) {
        OneDay oneDay = new XiaoMingDay();
        OneDay oneDay1 = new XiaoWangDay();
        oneDay.one();
        oneDay1.one();
}
```



##### 观察者模式

Spring 事件驱动模型就是观察者模式很经典的一个应用。

```java

//注册一个事件
public class QuestionEvent extends ApplicationEvent {
    private String question;
    public QuestionEvent(String question) {
        super(question);
        this.question = question;
    }
    public String getQuestion() {
        return question;
    }
    public void setQuestion(String question) {
        this.question = question;
    }
}

// 注册一个观察器
@Component
public class QuestionListener implements ApplicationListener<QuestionEvent> {
    @Override
    public void onApplicationEvent(QuestionEvent questionEvent) {
        System.out.println("QuestionListener......................" + questionEvent.toString());
    }
}

// 注册一个发布者
@Component
public class EventPublisher implements ApplicationEventPublisherAware {

    private ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher =applicationEventPublisher;
    }

    public ApplicationEventPublisher getApplicationEventPublisher() {
        return applicationEventPublisher;
    }
}

// 发布消息
@Test
public void testEvent() {
    String question = "这是一个问题";
    applicationEventPublisher.publishEvent(new QuestionEvent(question));
}
```


##### 策略模式

代码例子：

```java
//父类
public abstract class Strategy {
    public abstract void doSomething();
}

// 实现类
public class StrategyA extends Strategy {
    @Override
    public void doSomething() {
        System.out.println("A.......");
    }
}

public class StrategyB extends Strategy {
    @Override
    public void doSomething() {
        System.out.println("B.......");
    }
}
// 策略类
public class Context  {
    private Strategy strategy;
    public Context(Strategy strategy) {
        this.strategy = strategy;
    }
    public void contextDoSomething() {
        strategy.doSomething();
    }
}
// 调用
@Test
public void testEvent() {
    Context contextA = new Context(new StrategyA());
    Context contextB = new Context(new StrategyB());
    contextA.contextDoSomething();
    contextB.contextDoSomething();
}
```