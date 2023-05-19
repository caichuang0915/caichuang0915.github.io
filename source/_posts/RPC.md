---
title: 手写RPC
tags:
  - rpc
---

### 手写RPC

手动编写一个RPC框架，思路如下：

- 主要角色：使用者、服务提供端、服务消费端、注册中心（zookeeper）
- 实现目标：使用者引入我们的jar包，启动服务之后，需要作为生产者启动一个等待连接的server端口，同时也要作为一个消费者去获取其它的生产者的信息并调用
- 服务提供端的流程:
    - 获取所有需要暴露的接口。
    - 将需要暴露的接口注册到注册中心。
    - 启动sockerserver等待连接。
    - 添加编码、解码的handler。
- 服务消费端的流程:
    - 从注册中心拉取所有提供者的信息，保存到缓存中（redis、本地..）
    - 需要监听注册中心的变化，保证缓存中数据的有效性。
    - 找到所有需要远程调用的接口，并且为其生成代理类（实现发送请求和接收返回数据）。


<!-- more -->

以下记录下实现的关键步骤：

#### 服务端启动

服务端启动只需要做两件事，注册和启动socket监听端口。

```java
// 服务注册
public void run() throws Exception {
    zkRegistry.serviceRegistry();
    // 启动netty服务
    rpcServer.start();

}
```


#### 获取所有需要暴露的接口

自定义一个注解（RegistryAnno），使用者在接口实现上使用了该注解，则认为该接口需要暴露，并且将其注册到zk。

```java
public void serviceRegistry() throws Exception {
    // 获取服务中所有的 RegistryAnno 标注的类
    Map<String, Object> beansByAnno = SpringBeanUtil.getBeansByAnno(RegistryAnno.class);
    if(beansByAnno.isEmpty()){
        log.info("无需要注册的服务");
        return;
    }
    // 创建根节点
    serverZk.createRootIfNotExist();
    String ip = "127.0.0.1";
    for (Map.Entry<String, Object> entry : beansByAnno.entrySet()) {
        Object bean = entry.getValue();
        RegistryAnno annotation = bean.getClass().getAnnotation(RegistryAnno.class);
        Class<?> aClass = annotation.interfaceClass();
        // 创建接口节点
        serverZk.createIfNotExistPre(aClass.getName());
        // 创建提供者信息节点
        String node = aClass.getName() + "/" + ip + ":" + rpcProperty.getPort();
        serverZk.createIfNotExist(node);
        log.info("{}服务注册成功 提供者为 {}",aClass.getName(),node);
    }
}
```

#### 启动RPC服务端

- 采用Reactor 1+M+N 的线程模型。
- 此处需要注意handler的顺序，head在出去方向，tail在进入方向，addList则是在head后面的handler链上继续添加。
- handler一次解码和二次编码都是使用netty自带的编解码器，以消息开始的4个字节记录消息的长度，二次解码则是将bytebuf转换为java对象，一次编码则是将java对象转换为bytebuf。


```java
public void start() {
    EventLoopGroup boss = new NioEventLoopGroup(1,new DefaultThreadFactory("boss"));
    EventLoopGroup worker = new NioEventLoopGroup(0,new DefaultThreadFactory("worker"));
    EventExecutorGroup business = new NioEventLoopGroup(NettyRuntime.availableProcessors() * 2,new DefaultThreadFactory("business"));
    try {
        ServerBootstrap bootstrap  = new ServerBootstrap();
        bootstrap.group(boss,worker).channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG,1024)
                .option(ChannelOption.TCP_NODELAY,true)
                .option(ChannelOption.SO_KEEPALIVE,true)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        ChannelPipeline pipeline = socketChannel.pipeline();
                        // 返回数据二次编码
                        pipeline.addLast("SecondEnc",new LengthFieldPrepender(4));
                        pipeline.addLast("firstEnc",new FirstEnc());

                        // 接受到数据一次解码
                        pipeline.addLast("firstDec",new LengthFieldBasedFrameDecoder(Integer.MAX_VALUE,0, 4, 0, 4));
                        pipeline.addLast("secondDec",new SecondDec());

                        // 处理请求
                        pipeline.addLast(business,"request",new RequestChannelHandler());
                    }
                })
        ;
        ChannelFuture future = bootstrap.bind(rpcProperty.getPort()).syncUninterruptibly();
        log.info("netty server 启动成功");
        future.channel().closeFuture().syncUninterruptibly();
    } catch (InterruptedException e) {
        log.error("",e);
    } catch (Exception exception) {
        exception.printStackTrace();
    } finally {
        boss.shutdownGracefully();
        worker.shutdownGracefully();
        business.shutdownGracefully();
    }
}
```

#### 服务消费端启动

服务消费端启动只需要拉取注册中心的服务列表。

```java
public void discover() throws Exception {
    List<String> serviceList = serverZk.getServiceList();
    log.info("获取到的节点信息 {}", JSON.toJSONString(serviceList));
    if(CollectionUtils.isEmpty(serviceList)){
        return;
    }
    for (String s : serviceList) {
        serverZk.subscribeZKEvent(s);
        List<ServiceProvider> serviceInfos = serverZk.getServiceInfos(s);
        // 将信息保存在缓存（本地缓存、redis等，此处简单使用全局对象）中
        RegistryInfo.registryMap.put(s,serviceInfos);
    }
}
```

#### 生成代理类

- 使用者在外部接口时，此时容器中是没有接口对象的，需要在使用者在controller、service等添加容器时就生成代理对象并赋值。
- 使用postprocessor后置处理器中实现。
- 自定义一个注解（DiscoveryAnno），使用者在使用外部接口时标记该接口是外部的接口。
- 此处使用cglib代理。


```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    Field[] fields = bean.getClass().getDeclaredFields();
    for (Field field : fields) {
        if(!field.isAccessible()){
            field.setAccessible(true);
        }
        DiscoveryAnno annotation = field.getAnnotation(DiscoveryAnno.class);
        if(null==annotation){
            continue;
        }
        // 生成代理对象
        Class<?> type = field.getType();
        Object proxyInstance = requestProxy.createProxyObject(type);
        if(null!=proxyInstance){
            try {
                field.set(bean,proxyInstance);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }
    return bean;
}


public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
    if(ReflectionUtils.isObjectMethod(method)){
        return method.invoke(method.getDeclaringClass().newInstance(),objects);
    }
    RpcRequest rpcRequest = new RpcRequest();
    rpcRequest.setRequestId(UUID.randomUUID().toString());
    rpcRequest.setClassName(method.getDeclaringClass().getName());
    rpcRequest.setMethodName(method.getName());
    rpcRequest.setParameterTypes(method.getParameterTypes());
    rpcRequest.setParameters(objects);
    // 发送请求
    RequestManage requestManage = SpringBeanUtil.getBean(RequestManage.class);
    // 发送请求 获取响应
    RpcResponse rpcResponse = requestManage.sendRequest(rpcRequest);
    if(null==rpcResponse){
        return null;
    }
    return rpcResponse.getResult();
}
```


#### 发送请求

- 发送请求时需要从本地缓存中获取到提供方的信息，IP，端口等，用netty进行连接，并发送消息。
- 需要注意返回消息时，需要阻塞等待消息返回，并且需要区分返回的消息属于哪一个请求（此处使用promise + requestId实现）
- 发送请求时handler的顺序需要注意，此处与提供者不一样，head在出去server方向，tail在自己方向。


```java
public RpcResponse sendRequest(RpcRequest rpcRequest) throws Exception {
        List<ServiceProvider> serviceProviders = RegistryInfo.registryMap.get(rpcRequest.getClassName());
        if(CollectionUtils.isEmpty(serviceProviders)){
            throw new Exception();
        }
        ServiceProvider serviceProvider = serviceProviders.get(0);
        return nettySend(serviceProvider, rpcRequest);
    }


    private RpcResponse nettySend(ServiceProvider serviceProvider,RpcRequest rpcRequest) throws InterruptedException, ExecutionException {

        EventLoopGroup eventLoopGroup = new NioEventLoopGroup(0,new DefaultThreadFactory("client"));
        Bootstrap bootstrap = new Bootstrap();
        ResponselHandler responselHandler = new ResponselHandler();

        bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>(){

                    @Override
                    protected void initChannel(SocketChannel channel) throws Exception {
                        ChannelPipeline pipeline = channel.pipeline();


                        pipeline.addLast("secondtEnc1",new SecondEnc());
                        pipeline.addLast("firstEnc1",new SecondEncResp());

                        pipeline.addLast("secondDec1",new FirstDec());
                        pipeline.addLast("firstDec1",new FirstDecResp());

                        pipeline.addLast("hannler1",responselHandler);

                    }
                });
        ChannelFuture future = bootstrap.connect(serviceProvider.getServerIp(), serviceProvider.getRpcPort()).sync();
        if(future.isSuccess()){

            Channel channel = future.channel();

            DefaultPromise defaultPromise = new DefaultPromise(channel.eventLoop());

            RegistryInfo.promiseMap.put(rpcRequest.getRequestId(),defaultPromise);

            channel.writeAndFlush(rpcRequest);

            RpcResponse rpcResponse = (RpcResponse) defaultPromise.get();

            return rpcResponse;

        }

        return new RpcResponse();

    }
```

#### 使用者

- 提供方需要标明需要暴露的接口

```java
@Service
@RegistryAnno(interfaceClass = TService.class)
public class TServiceImpl implements TService {

    @Override
    public String doTest() {
        return "doTest";
    }
}
```

- 消费方需要标明外部接口

```java
@RestController
public class MainController {

    @DiscoveryAnno
    private TService tService;


    @RequestMapping("/first")
    public String rpcTest() {
        return tService.doTest();
    }
}
```