---
title: RabbitMq - 入门
tags:
  - RabbitMq
---

AMQP概论

## 交换器、队列、绑定、路由键

- rabbitMq流程

![rabbitMq流程](http://image.tupelo.top/rabbitmq%E6%B5%81%E7%A8%8B.png)

	交换机类型:
	fanout 交换机绑定队列,根据交换机发送消息到指定队列
	direct 交换机通过路由键绑定队列,根据交换机和路由键发送消息
	topic  交换机通过路由键绑定队列,根据交换机和路由键发送消息(路由键可进行匹配 #匹配一次或者多次 *匹配一次)

> 需要多个队列接受同样的消息 多个队列绑定同一个路由键 一条消息只能被一个消费者消费

<!-- more -->

### 不同交换机的Demo

- 添加依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

- 创建配置类,初始化的时候指定交换机的类型

```java

@Configuration
public class RabbitMqConfig {

	// 创建连接
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setHost(host);
        connectionFactory.setPort(port);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost(virtualHost);
        return connectionFactory;
    }

    // 初始化 RabbitTemplate
    @Bean
    @Primary
    public RabbitTemplate rabbitTemplate() {
        return new RabbitTemplate(connectionFactory());
    }

    // 创建队列 没有的话会创建新的队列
    @Bean
    public Queue fanoutQueue(){
        return new Queue("two.queue");
    }
    @Bean
    public Queue directQueue(){
        return new Queue("four.queue");
    }
    @Bean
    public Queue topQueue(){
        return new Queue("three.queue");
    }
    @Bean
    public Queue directOneQueue(){
        return new Queue("four_one.queue");
    }

    // 创建三种类型的交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("two.exchange");
    }
    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange("four.exchange");
    }
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange("three.exchange");
    }

    // 交换机和队列绑定
    @Bean
    public Binding fanoutBinding(){
        return BindingBuilder.bind(fanoutQueue()).to(fanoutExchange());
    }
    @Bean
    public Binding directBinding(){
        return BindingBuilder.bind(directQueue()).to(directExchange()).with("four.bind");
    }
    @Bean
    public Binding directOneBinding(){
        return BindingBuilder.bind(directOneQueue()).to(directExchange()).with("four.bind");
    }
    @Bean
    public Binding topicBinding(){
        return BindingBuilder.bind(topQueue()).to(topicExchange()).with("topicBind");
    }
}
```

- 消费者部分代码,只需要监听指定队列,获取的消息是根据生产者发送过来的消息(Message/String)

```java
@RabbitListener(queues = "four.queue")
@RabbitHandler
public void fanoutConsumer2(Object msg){
    Message message = (Message) msg;
    log.info("2fanoutConsumer 接收消息msg: {}" , new String(message.getBody()));
}

@RabbitListener(queues = "four.queue")
@RabbitHandler
public void fanoutConsumer3(String msg){
    log.info("3fanoutConsumer 接收消息msg: {}" , msg);
}
```

- 生产者部分代码(fanout方式路由键参数不生效),生产者不用考虑队列,投放消息只和交换机和路由键相关

```java
// 发送Message
@RequestMapping("/send1/{msg}")
public String sendMessage(@PathVariable("msg") String msg) {
    log.info("msg1");
    Message message = new Message(msg.getBytes(),new MessageProperties());
    rabbitTemplate.send("four.exchange","aqweq",message);
    return msg;
}
// 发送String
@RequestMapping("/send/{msg}")
public String sendMessage1(@PathVariable("msg") String msg) {
   log.info("msg2");
   amqpTemplate.convertAndSend("four.exchange","four.bind",msg);
   return msg;
}
```












