---
title: RabbitMq - 生产者消息确认
tags:
  - RabbitMq
---

RabbitMq生产者消息确认

## RabbitMq 生产者消息确认

### 生产者权衡
> 快、低==========> 慢、高

- 无保障
- 失败通知  参数mandatory 消息是否成功到达队列
- 发送方确认模式  消息是否成功到达exchange 和失败通知结合使用
- 备用交换机     定义个一个备用交换机 Map
- 高可用队列
- 事务          影响性能 2-10倍 强一致性
- 事务+高可用
- 持久化

<!-- more -->

#### 失败通知 需要开启 PublisherReturns=true mandatory=true

```java
// 初始化一个rabbitTemplateReturn 用来做失败通知 设置PublisherReturns=true
@Bean(value = "rabbitTemplateReturn")
public RabbitTemplate rabbitTemplateReturn() {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
    ((CachingConnectionFactory)rabbitTemplate.getConnectionFactory()).setPublisherReturns(true);
    return rabbitTemplate;
}

// 定义一个类实现RabbitTemplate.ReturnCallback 并且开启mandatory=true 指定ReturnCallback
@Component
public class RabbitReturnCallback implements RabbitTemplate.ReturnCallback {

    @Autowired
    @Qualifier("rabbitTemplateReturn")
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //指定 ReturnCallback
        rabbitTemplate.setReturnCallback(this);
        rabbitTemplate.setMandatory(true);
    }

    @Override
    public void returnedMessage(Message message, int i, String s, String s1, String s2) {
        System.out.println("消息主体 message : "+message);
        System.out.println("消息主体 message : "+i);
        System.out.println("描述："+s);
        System.out.println("消息使用的交换器 exchange : "+s1);
        System.out.println("消息使用的路由键 routing : "+s2);
    }
}

// 发送方 若没有成功路由 则回调returnedMessage方法
@RequestMapping("/send2/{msg}")
public String sendMessageReturn(@PathVariable("msg") String msg) {
    log.info("msg1");
    Message message = new Message(msg.getBytes(),new MessageProperties());
    rabbitTemplateRerutn.send("four.exchange","aqweq",message);
    return msg;
}

```

#### 发送方确认模式 需要开启 PublisherReturns=true mandatory=true

	确认的三种方式:  
	channel.waitForConfirms()普通发送方确认模式；消息到达交换器，就会返回true  
	channel.waitForConfirmsOrDie()批量确认模式；使用同步方式等所有的消息发送之后才会执行后面代码，只要有一个消息未到达交换器就会抛出IOException异常  
	channel.addConfirmListener()异步监听发送方确认模式  

```java

// 初始化一个rabbitTemplateConfirm 用来做失败通知 设置PublisherReturns=true
@Bean(value = "rabbitTemplateConfirm")
public RabbitTemplate rabbitTemplateConfirm() {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
    ((CachingConnectionFactory)rabbitTemplate.getConnectionFactory()).setPublisherConfirms(true);
    return rabbitTemplate;
}


/**
*	定义一个类实现RabbitTemplate.ReturnCallback 并且开启mandatory=true 指定ConfirmCallback
*	ack 消息是否成功到达exchange 若没有正确路由到队列 ack返回也是true 所以和失败通知结合使用 
*   这是异步处理的方式
*/
@Component
public class RabbitConfirmCallback implements RabbitTemplate.ConfirmCallback {

    @Autowired
    @Qualifier("rabbitTemplateConfirm")
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //指定 ConfirmCallback
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setMandatory(true);
    }

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause){
        System.out.println("消息唯一标识："+correlationData);
        System.out.println("确认结果："+ack);
        System.out.println("失败原因："+cause);
    }
}

// 发送方 
@RequestMapping("/send3/{msg}")
public String sendMessageConfirm(@PathVariable("msg") String msg) {
    String msgId = UUID.randomUUID().toString();
	CorrelationData date = new CorrelationData(msgId);
	// message 可以设置一些头信息 msgId 
	Message message = MessageBuilder.withBody("测试".toString().getBytes())
                .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
                .setCorrelationId(msgId).build();
    rabbitTemplateConfirm.send("four.exchange1","aqweq",message,date);
    return msg;
}


/**
*  
*   同步处理方式如下：
*   发送方
*   TODO operations.waitForConfirms返回一直是true? 待处理
*/

@RequestMapping("/send4/{msg}")
public String sendMessageOperateConfirm(@PathVariable("msg") String msg) {
    Message message = new Message(msg.getBytes(),new MessageProperties());
    rabbitTemplateOperateConfirm.send("four.exchange1","aqweq",message);

    rabbitTemplateOperateConfirm.invoke(new RabbitOperations.OperationsCallback<Object>() {
        @Override
        public Object doInRabbit(RabbitOperations operations) {
//          operations.waitForConfirmsOrDie(100);
            System.out.println(operations.waitForConfirms(100));
            return null;
        }
    });
    return msg;
}

```

> 以上确认可互相配合

- 发送消息前,绑定并保存msgId和message的关系
- 当confirm或return回调时,根据ack类别等,分别处理. 例如return或者ack=false则说明有问题,报警, 如果ack=true则删除关系(因为return在confirm前,所以一条消息在return后又ack=true的情况也是按return处理)
- 定时检查这个绑定关系列表,如果发现一些已经超时(自己设定的超时时间)未被处理(即未return和confirm),则手动处理这些消息.
- 需要注意如果是自动重发的话,消费端需要做幂等或去重处理.



#### 备用交换机

- 定义一个备用交换机
- 声明一个map key为"alternate-exchange" value为备用交换机的名称
- 声明主交换机时添加参数map
- 如果发送消息到主交换机失败则会转发到备用交换机

```java

// 定义备用交换机 一般为fanout类型 路由键一般为#
@Bean
public Queue fanoutBakQueue(){
    return new Queue("four_bak.queue");
}
@Bean
public FanoutExchange fanoutBakExchange(){
    return new FanoutExchange("four.exchange_bak");
}
@Bean
public Binding fanoutBakBinding(){
    return BindingBuilder.bind(fanoutBakQueue()).to(fanoutBakExchange());
}

// 定义主交换机 定义map
@Bean
public DirectExchange directExchange(){
    Map<String,Object> map = new HashMap<String,Object>(16);
    map.put("alternate-exchange","four.exchange_bak");
    return new DirectExchange("four.exchange",false,false,map);
}

```

#### 高可用队列

> 集群情况使用

#### 消息持久化

> 可靠性最高


#### 事务 事务高可用队列

> 不使用 性能损耗严重







