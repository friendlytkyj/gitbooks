@[TOC](目录)

# Spring Boot整合RabbitMQ由浅入深完整讲解

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
- AmqpAdmin队列过滤规则

## 配置单个RabbitMQ

### 先来一个最最简单的示例

在`application.properties`配置文件中增加如下配置

```
spring.rabbitmq.host: 127.0.0.1
spring.rabbitmq.port: 5672
spring.rabbitmq.username: guest
spring.rabbitmq.password: guest
```

创建一个配置类`RabbitmqConfiguration.java`，并且在这个类的上面使用`@Configuration`注解，我们用这个配置类来初始化MQ相关的配置

```
@Configuration
public class RabbitmqConfiguration {

}
```

### 声明队列

```
	@Bean
	public Queue mqQueue1() {
		String queueName = "mq-queue-1";
		return new Queue(queueName, true, false, false, new HashMap<>());
	}
```

### 声明交换机

```
	@Bean
	public Exchange mqExchage1() {
		String exchangeName = "mq-exchange-1";
        return new FanoutExchange(exchangeName, true, false, new HashMap<>());
    }
```

### 声明绑定

```
	@Bean
	public Binding bindQueue1AndExchange1(
			@Qualifier("mqQueue1") Queue mqQueue1,
			@Qualifier("mqExchage1") Exchange mqExchage1) {
		
        return BindingBuilder.bind(mqQueue1).to(mqExchage1).with("").noargs();
    }
```

### 定义放入MQ的消息数据结构

```
@Data
public class MqMessage {

	/**主键*/
    private String msgId;

	/**发送人*/
    private String from;
    /**接收人*/
    private String to;
    /**消息内容*/
    private String content;
    /**发送时间*/
    private long sendTime;
}
```

### 发布消息到RabbitMQ

**Spring Boot**框架在启动后就可以使用`@Autowired`方式使用`MongoTemplate`对象

```
private @Autowired RabbitTemplate rabbitTemplate;
```

发布消息

```
String exchange = "mq-exchange-1";
MqMessage mqMessage = buildMessage();
rabbitTemplate.convertAndSend(exchange, "", jsonUtil.toJson(mqMessage));
```

buildMessage()代码

```
	public MqMessage buildMessage() {
		MqMessage message = new MqMessage();
		
		message.setMsgId(UUID.randomUUID().toString());
		message.setFrom("张三");
		message.setTo("李四");
		message.setContent("hello, nice to meet you!");
		message.setSendTime(System.currentTimeMillis());
		
		return message;
	}
```

### 创建消费者

使用`@RabbitListener`和`@RabbitHandler`两个注解完成消费者的创建

- 在类的上面使用`@RabbitListener`，设置订阅的队列以及ack方式
- 在方法的上面使用`@RabbitHandler`
  > `@RabbitHandler`的方法最多可以接收4个参数，分别是
    - 放入RabbitMQ的消息数据，比如String，这个参数是必须要有的，另外三个参数非必须，根据业务场景自己添加
    - org.springframework.messaging.Message
    - org.springframework.amqp.core.Message
    - com.rabbitmq.client.Channel
- 为了方便写`Junit`测试，加了一个`List`存放消费结果

```
@Component
@RabbitListener(queues = "mq-queue-1", ackMode = SpringRabbitConsumer.ACK_MODE)
@Slf4j
public class SpringRabbitConsumer {
	
	public static final String ACK_MODE = "MANUAL";

	private List<MqMessage> mqMessageList = new ArrayList<>();
	private JsonUtil jsonUtil;
	
	@Autowired
	public SpringRabbitConsumer(JsonUtil jsonUtil) {
		this.jsonUtil = jsonUtil;
	}

	@RabbitHandler
	public void onMessage(String message, Message amqpMessage, Channel channel) throws Exception {
		long deliveryTag = amqpMessage.getMessageProperties().getDeliveryTag();
		
		log.debug("Spring Rabbit Consumer: {}", message);
		
		try {
			mqMessageList.add(jsonUtil.fromJson(message, MqMessage.class));
		} catch (Exception e) {
			log.error("===", e);
		} finally {
			channel.basicAck(deliveryTag, false);
		}
	}

	public List<MqMessage> getMqMessageList() {
		return mqMessageList;
	}
}
```

### 编写Junit单元测试

使用`RabbitTemplate`发布消息到**RabbitMQ**，然后消费者从**RabbitMQ**取出消息进行业务逻辑处理，我们简单的把取出来的消息放下一个`List`

```
	@Test
	public void test() throws Exception {
		String exchange = "mq-exchange-1";
		// 构造一条业务数据
		MqMessage mqMessage = buildMessage();
		// 发布业务数据到RabbitMQ
		rabbitTemplate.convertAndSend(exchange, "", jsonUtil.toJson(mqMessage));
		// sleep一下，等消费者消费数据
		ThreadUtils.sleep(DurationUtils.toDuration(1000, TimeUnit.MILLISECONDS));
		// 从消费者获取消费结果
		List<MqMessage> consumerResult = springRabbitConsumer.getMqMessageList();
		
		// 校验测试结果
		assertFalse(consumerResult.isEmpty());
		MqMessage lastConsume = consumerResult.get(consumerResult.size()-1);
		assertEquals(mqMessage.getMsgId(), lastConsume.getMsgId());
		
		log.debug("测试通过");
	}
```

以上就是**Spring Boot**整合**RabbitMQ**最简单的一个例子，**Spring Boot**已经帮我们做了很多事情，我们只需要做很少的配置，使用注解很快就可以完成对**RabbitMQ**的发布以及订阅。

## 配置多个RabbitMQ

只连一个**RabbitMQ**的时候非常简单，但在实际使用中可能需要连两个甚至两个以上的**RabbitMQ**，那要如何进行配置呢？我们接下来继续学习。

想要配置多个**RabbitMQ**，我们就需要了解更多**Spring Boot**相关的原理，学习一下**Spring Boot**到底帮我们做了哪些事情。

### 了解RabbitAutoConfiguration

`RabbitAutoConfiguration`这个配置类，默默的帮我们创建了很多东西，列一些比较重要的**Bean**
  - `CachingConnectionFactory`
  - `RabbitTemplate`
  - `AmqpAdmin`
  - `RabbitMessagingTemplate`
  - `SimpleRabbitListenerContainerFactoryConfigurer`
  - `SimpleRabbitListenerContainerFactory`

这些**Bean**的创建，是有前提条件的，有些是`@ConditionalOnMissingBean`，有些是`@ConditionalOnSingleCandidate`……

为什么要了解**RabbitAutoConfiguration**这个类呢？因为配置多个**RabbitMQ**时会影响到**Spring Boot**的这些默认行为，同时我们也可以从这个类中学习如何构造需要的东西，这是进入高级应用的学习入口。

### 多个RabbitMQ配置

我们以两个**RabbitMQ**为例，一个以**default**命名，一个以**second**命名。

在配置文件中追加配置如下

```
spring.rabbitmq.second.host: 127.0.0.1
spring.rabbitmq.second.port: 5672
spring.rabbitmq.second.username: guest
spring.rabbitmq.second.password: guest
```

### 继续改造RabbitmqConfiguration.java

上面已经在`RabbitmqConfiguration`类中声明了队列、交换机以及绑定关系

如果要连多个**RabbitMQ**，就需要手动创建`CachingConnectionFactory`、`RabbitTemplate`

### 读取second配置

```
@Configuration
@ConfigurationProperties(prefix = "spring.rabbitmq")
public class RabbitmqConfiguration {

	// springboot创建的RabbitProperties
	@Qualifier("spring.rabbitmq-org.springframework.boot.autoconfigure.amqp.RabbitProperties")
	private @Autowired RabbitProperties defaultRabbitProperties;

	// 第2个RabbitMQ的配置
	private RabbitProperties second;
	
	public void setSecond(RabbitProperties second) {
		this.second = second;
	}
}
```

### 创建队列、交换机、绑定

根据`mqQueue1`的创建方式，再创建一个`mqQueue2`

```
	@Bean
	public Queue mqQueue2() {
		String queueName = "mq-queue-2";
		return new Queue(queueName, true, false, false, new HashMap<>());
	}
	
	@Bean
	public Exchange mqExchage2() {
		String exchangeName = "mq-exchange-2";
        return new FanoutExchange(exchangeName, true, false, new HashMap<>());
    }
	
	@Bean
	public Binding bindQueue2AndExchange2(
			@Qualifier("mqQueue2") Queue mqQueue2,
			@Qualifier("mqExchage2") Exchange mqExchage2) {
		
        return BindingBuilder.bind(mqQueue2).to(mqExchage2).with("").noargs();
    }
```

### 创建ConnectionFactory

```
	@Bean
    public ConnectionFactory connectionFactory() {
        return createConnectionFactory(defaultRabbitProperties);
    }
	
	@Bean
    public ConnectionFactory secondConnectionFactory() {
        return createConnectionFactory(second);
    }
```

createConnectionFactory方法的代码如下

```
	private ConnectionFactory createConnectionFactory(RabbitProperties properties){
		CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setHost(properties.getHost());
        connectionFactory.setPort(properties.getPort());
        connectionFactory.setUsername(properties.getUsername());
        connectionFactory.setPassword(properties.getPassword());
        return connectionFactory;
    }
```

### 创建RabbitTemplate

```
	@Bean
	public RabbitTemplate defaultRabbitTemplate(@Qualifier("connectionFactory") ConnectionFactory connectionFactory) {
		RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
		return rabbitTemplate;
	}
	
	@Bean
	public RabbitTemplate secondRabbitTemplate(@Qualifier("secondConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
		return rabbitTemplate;
	}
```

### 编写Junit单元测试



### 报错

```
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'rabbitListenerContainerFactory' defined in class path resource [org/springframework/boot/autoconfigure/amqp/RabbitAnnotationDrivenConfiguration.class]: Unsatisfied dependency expressed through method 'simpleRabbitListenerContainerFactory' parameter 1; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.springframework.amqp.rabbit.connection.ConnectionFactory' available: expected single matching bean but found 2: defaultConnectionFactory,secondConnectionFactory
```

## 消费者两种实现方式

### 注解方式实现消费者
@RabbitListener+@RabbitHandler

### Container方式实现消费者

#### ChannelAwareMessageListener介绍


#### SimpleMessageListenerContainer介绍
