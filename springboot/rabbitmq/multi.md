@[TOC](目录)

# Spring Boot怎么实现配置多个RabbitMQ

> 本文需要读者对Spring Boot整合RabbitMQ有基本了解

## 环境介绍

|软件名称|软件版本|
|:--|:--|
|Spring Boot|2.5.3|
|Maven|3.6.3|
|RabbitMQ Server|3.9.1|
|erlang|24.0.5|

## 疑问

- 以@Bean的方式创建队列，如何在指定的MQ创建，Spring Boot是在什么时候创建队列，是使用默认的ConnectionFactory创建队列吗？
- 如何对不同的MQ创建各自不同的队列？
> org.springframework.amqp.rabbit.connection.ChannelListener
  在这个接口中实现队列的创建
- rabbitmq创建动态队列并订阅消费
业务场景: 程序启动后，根据业务规则动态生成队列名称，可能每次启动后生成的队列名称不相同
- 如何消费这个队列？
- @RabbitHandler注解定义的方法，参数如何声明？
> 方法参数，必须要有一个跟实际放入MQ的参数类型一致，比如放到MQ的是String类型的数据，那么就要有一个String类型的参数

## 配置单个RabbitMQ

在`application.properties`配置文件中增加如下配置

```
spring.data.mongodb.uri: mongodb://localhost:27017/Teach
```

**Spring Boot**框架在启动后就可以使用`@Autowired`方式使用`MongoTemplate`对象

```
private @Autowired MongoTemplate mongoTemplate;
```

`MongoTemplate`对象可以实现对**MongoDB**数据的**CRUD**操作

## 消费者两种实现方式

### 注解方式实现消费者
@RabbitListener+@RabbitHandler

### Container方式实现消费者

#### ChannelAwareMessageListener介绍


#### SimpleMessageListenerContainer介绍
