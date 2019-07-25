---
title: RabbitMq - 定义队列,消息的其它属性(定义时最后的一个Map)
tags:
  - RabbitMq
---

RabbitMq其它属性(定义时最后的一个Map)

## RabbitMq 其它属性(定义时最后的一个Map)

#### 队列

![队列](http://image.tupelo.top/queue.png)

<!-- more -->

```
@Bean
public Queue topicQueueDlxKeyBak(){
    Map<String,Object> map = new HashMap<String,Object>(16);
    // 定义属性 ..
    map.put("x-dead-letter-exchange","dlx.key.exchange");
    map.put("x-dead-letter-routing-key","dlx.key.bind");
    return new Queue("dlx.key.queue",false,false,false,map);
}
```

#### 消息

![消息](http://image.tupelo.top/rabbitMessage.png)


```
// 设置消息属性
Message message = MessageBuilder.withBody("测试".toString().getBytes())
                .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
                .setCorrelationId(msgId).build();
rabbitTemplateConfirm.send("exchange","routing",message);
```






