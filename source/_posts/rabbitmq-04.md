---
title: RabbitMq - 消费者消息确认
tags:
  - RabbitMq
---

RabbitMq消费者消息确认

## RabbitMq 消费者消息确认

### 消费者权衡

- 获取消息方式
    - 拉取get
    - 自动推送
- 消息应答方式
    - 自动确认
    - 手动确认
- Qos预取模式
- 可靠性
    - Qos 批量2500事务(高)
    - 拉取事务(低)
- 性能 
    - 总量2500Qos(高)
    - 少量Qos 批量2500事务
    - 事务
    - 拉取(低)

<!-- more -->


#### 获取消息方式

##### 拉取get

> channel.basicGet(String queueName, boolean autoAck) 效率低不常用

##### 自动推送

> springBoot监听队列注解:@RabbitListener(queues = "queueName"),处理方法注解:@RabbitHandler,默认是自动确认。(注解情况下手动确认?)

#### 消息应答方式、Qos模式、批量确认、单条确认

```java

@Bean
public SimpleMessageListenerContainer messageListenerContainer(){
    SimpleMessageListenerContainer messageListenerContainer = new SimpleMessageListenerContainer(connectionFactory());
    // 开启Qos
    //messageListenerContainer.setPrefetchCount(2500);
    messageListenerContainer.setQueues(topicQueueBak());
    // 开启手动确认 MANUAL  
    messageListenerContainer.setAcknowledgeMode(AcknowledgeMode.MANUAL);
    messageListenerContainer.setMessageListener(firstConsumer);
    return messageListenerContainer;
}

// 实现手动确认 实现ChannelAwareMessageListener接口，basicAck方法为成功确认

@Component
public class FirstConsumer implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        try {
            String msg = new String(message.getBody());
            System.out.println("ack>>>>>>>接收到消息:"+msg);
            try {
                channel.basicAck(message.getMessageProperties().getDeliveryTag(),
                        false);
                System.out.println("ack>>>>>>消息已消费");
            } catch (Exception e) {
                // 第一false 是否批量拒绝 第二个是否重新放入队列
                channel.basicNack(message.getMessageProperties().getDeliveryTag(),
                        false,false);
                System.out.println("UserReceiver>>>>>>拒绝消息，要求Mq重新派发");
                throw e;
            }

        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

#### 消息的拒绝

> Nack 可以批量拒绝 第一false是否批量拒绝 第二个是否重新放入队列 重新放入队列容易产生死循环  

    channel.basicNack(message.getMessageProperties().getDeliveryTag(),
                        false,false);

> Reject 单条拒绝  

    channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);

#### 死信交换器DLX(消息生产者的备用交换器类似)

> 将拒绝的消息可以投放至自定义的队列处理,重新定义一个队列绑定主交换机,设置参数"x-dead-letter-exchange",也可以添加死信路由"x-dead-letter-routing-key",这样不同业务拒绝的消息可以投递至不同的队列。

```java

// 死信交换器
@Bean
public Queue topicQueueDlxBak(){
    Map<String,Object> map = new HashMap<String,Object>(16);
    // 死信交换器
    map.put("x-dead-letter-exchange","dlx.exchange");
    return new Queue("dlx.queue",false,false,false,map);
}
@Bean
public TopicExchange topicExchangeDlxBak(){
    return new TopicExchange("dlx.exchange");
}
@Bean
public Binding topicBindingDlxBak(){
    return BindingBuilder.bind(topicQueueDlxBak()).to(topicExchangeBak()).with("#");
}

// 死信交换器 + 死信路由
@Bean
public Queue topicQueueDlxKeyBak(){
    Map<String,Object> map = new HashMap<String,Object>(16);
    map.put("x-dead-letter-exchange","dlx.key.exchange");
    map.put("x-dead-letter-routing-key","dlx.key.bind");
    return new Queue("dlx.key.queue",false,false,false,map);
}
@Bean
public TopicExchange topicExchangeDlxKeyBak(){
    return new TopicExchange("dlx.key.exchange");
}
@Bean
public Binding topicBindingDlxKeyBak(){
    return BindingBuilder.bind(topicQueueDlxKeyBak()).to(topicExchangeBak()).with("dlx.key.bind");
}
```



