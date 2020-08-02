---
title: SpringIOC - IOC源码分析
tags:
  - Java
  - Spring
---

IOC源码分析

### IOC流程

#### IOC源码入口

```java

	// Spring启动入口
	AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(Cap12MainConfig.class);
	app.close();


	// Spring容器的refresh()【创建刷新】
	public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
```

<!-- more -->


#### IOC流程详解

Spring容器的refresh()【创建刷新】;

* prepareRefresh() 刷新前的预处理
	- initPropertySources() 初始化一些属性设置;子类自定义个性化的属性设置方法
	- getEnvironment().validateRequiredProperties() 检验属性的合法等
	- earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>() 保存容器中的一些早期的事件

* obtainFreshBeanFactory() 获取BeanFactory
	- refreshBeanFactory();刷新【创建】BeanFactory;110行:创建了一个this.beanFactory = new DefaultListableBeanFactory();设置id；
	- getBeanFactory();返回刚才GenericApplicationContext创建的BeanFactory对象；
	- 将创建的BeanFactory【DefaultListableBeanFactory】返回；

* prepareBeanFactory(beanFactory) BeanFactory的预准备工作（以上创建了beanFactory,现在对BeanFactory对象进行一些设置属性）
	- 设置BeanFactory的类加载器、支持表达式解析器...
	- 添加部分BeanPostProcessor【ApplicationContextAwareProcessor】
	- 设置忽略的自动装配的接口EnvironmentAware、EmbeddedValueResolverAware、xxx；
	- 注册可以解析的自动装配；我们能直接在任何组件中自动注入:BeanFactory、ResourceLoader、ApplicationEventPublisher、ApplicationContext
	- 添加BeanPostProcessor【ApplicationListenerDetector】
	- 添加编译时的AspectJ；
	- 给BeanFactory中注册一些能用的组件; 
		environment【ConfigurableEnvironment】、systemProperties【Map<String, Object>】、systemEnvironment【Map<String, Object>】

* postProcessBeanFactory(beanFactory) BeanFactory准备工作完成后进行的后置处理工作
	- 子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置


> 以上是BeanFactory的创建及预准备工作

---


* invokeBeanFactoryPostProcessors(beanFactory) 执行BeanFactoryPostProcessor的方法

	> BeanFactoryPostProcessor:BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的,两个接口:BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor

	- 执行BeanFactoryPostProcessor的方法
		先执行BeanDefinitionRegistryPostProcessor
		- 83行:获取所有的BeanDefinitionRegistryPostProcessor；
		- 86行:看先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor、
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		- 99行:在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
		- 109行:最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessors；
			postProcessor.postProcessBeanDefinitionRegistry(registry)
	- 再执行BeanFactoryPostProcessor的方法
		- 139行:获取所有的BeanFactoryPostProcessor
		- 147行:看先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor、
			postProcessor.postProcessBeanFactory()
		- 167行:在执行实现了Ordered顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()
		- 175行:最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor；
			postProcessor.postProcessBeanFactory()


* registerBeanPostProcessors(beanFactory) 注册BeanPostProcessor（Bean的后置处理器）
	```
	不同接口类型的BeanPostProcessor；在Bean创建前后的执行时机是不一样的
	BeanPostProcessor、
	DestructionAwareBeanPostProcessor、
	InstantiationAwareBeanPostProcessor、
	SmartInstantiationAwareBeanPostProcessor、
	MergedBeanDefinitionPostProcessor【internalPostProcessors】
	```
	- 189行:获取所有的BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来执行优先级
	- 204行:先注册PriorityOrdered优先级接口的BeanPostProcessor,把每一个BeanPostProcessor添加到BeanFactory中,beanFactory.addBeanPostProcessor(postProcessor);
	- 224行:再注册Ordered接口的
	- 236行:最后注册没有实现任何优先级接口的
	- 最终注册MergedBeanDefinitionPostProcessor；
	- 注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是:applicationContext.addApplicationListener((ApplicationListener<?>) bean);


* initMessageSource() 初始化MessageSource组件（做国际化功能；消息绑定，消息解析）

	- 718行:获取BeanFactory
	- 719行:看容器中是否有id为messageSource的，类型是MessageSource的组件,如果有赋值给messageSource，如果没有自己创建一个DelegatingMessageSource;MessageSource:取出国际化配置文件中的某个key的值,能按照区域信息获取
	- 739行:把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource；

	```java
	beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
	MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);
	//以后可通过getMessage获取 
	```
	


* initApplicationEventMulticaster() 初始化事件派发器

	- 753行:获取BeanFactory
	- 754行:从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster
	- 762行:如果上一步没有配置；创建一个SimpleApplicationEventMulticaster
	- 763行:将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入


* onRefresh() 留给子容器（子类）

	- 子类重写这个方法，在容器刷新的时候可以自定义逻辑；


* registerListeners() 给容器中将所有项目里面的ApplicationListener注册进来

	- 822行:从容器中拿到所有的ApplicationListener
	- 824行:将每个监听器添加到事件派发器中
	```java
	getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName)
	```
	- 832行:派发之前步骤产生的事件；


* finishBeanFactoryInitialization(beanFactory) 初始化所有剩下的单实例bean 

	- 867行:beanFactory.preInstantiateSingletons();初始化后剩下的单实例bean，跟进
		- 734行:获取容器中的所有Bean，依次进行初始化和创建对象
		- 738行:获取Bean的定义信息；RootBeanDefinition
		- 739行:Bean不是抽象的，是单实例的，是懒加载；
			- 740行:判断是否是FactoryBean；是否是实现FactoryBean接口的Bean；
			- 760行:不是工厂Bean。利用getBean(beanName);创建对象
				- 199行:getBean(beanName)； ioc.getBean();
				- doGetBean(name, null, null, false);
				- 246行: getSingleton(beanName)先获取缓存中保存的单实例Bean《跟进去其实就是从MAP中拿》。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）
				```java
				// 从这个中获取的
				private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
				```
				- 缓存中获取不到，开始Bean的创建对象流程；
				- 287行:标记当前bean已经被创建（防止多线程同时创建，使用synchronized）
				- 291行:获取Bean的定义信息；
				- 295行:getDependsOn()，bean.xml里创建person时，加depend-on="jeep,moon"是先把jeep和moon创建出来,【获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来；】
				- 启动单实例Bean的创建流程；
					- 462行:createBean(beanName, mbd, args);
					- 490行:Object bean = resolveBeforeInstantiation(beanName, mbdToUse);让BeanPostProcessor先拦截返回代理对象；
						【InstantiationAwareBeanPostProcessor】:提前执行；
						先触发:postProcessBeforeInstantiation()；
						如果有返回值:触发postProcessAfterInitialization()；
					- 如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用下面
					- 501行:Object beanInstance = doCreateBean(beanName, mbdToUse, args);创建Bean
						 - 541行:【创建Bean实例】；createBeanInstance(beanName, mbd, args);
						 	利用工厂方法或者对象的构造器创建出Bean实例；
						 - applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
						 	调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition(mbd, beanType, beanName);
						 - 578行:【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
						 	赋值之前:
						 	- 拿到InstantiationAwareBeanPostProcessor后置处理器；
						 		1305行:postProcessAfterInstantiation()；
						 	- 拿到InstantiationAwareBeanPostProcessor后置处理器；
						 		1348行:postProcessPropertyValues()；
						 	=====赋值之前:===
						 	- 应用Bean属性的值；为属性利用setter方法等进行赋值；
						 		applyPropertyValues(beanName, mbd, bw, pvs);
						 - 【Bean初始化】initializeBean(beanName, exposedObject, mbd);
						 	- 1693行:【执行Aware接口方法】invokeAwareMethods(beanName, bean);执行xxxAware接口的方法
						 		BeanNameAware\BeanClassLoaderAware\BeanFactoryAware
						 	- 1698行:【执行后置处理器初始化之前】applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
						 		BeanPostProcessor.postProcessBeforeInitialization（）;
						 	- 1702行:【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);
						 		- 是否是InitializingBean接口的实现；执行接口规定的初始化；
						 		- 是否自定义初始化方法；
						 	- 1710行:【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization
						 		BeanPostProcessor.postProcessAfterInitialization()；
						
					- 将创建的Bean添加到缓存中singletonObjects；sharedInstance = getSingleton(beanName, ()跟进去
					     254行:addSingleton（），放到MAP中
				ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。。；
		所有Bean都利用getBean创建完成以后；
			检查所有的Bean是否是SmartInitializingSingleton接口的；如果是；就执行afterSingletonsInstantiated()；


* finishRefresh() 完成BeanFactory的初始化创建工作,IOC容器就创建完成

	- 882行:initLifecycleProcessor() 初始化和生命周期有关的后置处理器:LifecycleProcessor,默认从容器中找是否有lifecycleProcessor的组件【LifecycleProcessor】;如果没有new DefaultLifecycleProcessor()加入到容器；
	```
	自己也可以尝试写一个LifecycleProcessor的实现类，可以在BeanFactory
		void onRefresh();
		void onClose();	
	```
	- 885行:getLifecycleProcessor().onRefresh() 拿到前面定义的生命周期处理器（BeanFactory） 回调onRefresh()
	- 888行:publishEvent(new ContextRefreshedEvent(this));发布容器刷新完成事件；
	- 891行:liveBeansView.registerApplicationContext(this);
	
