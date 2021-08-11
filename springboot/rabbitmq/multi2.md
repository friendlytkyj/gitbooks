@[TOC](目录)

# Spring Boot整合RabbitMQ由浅入深完整讲解(二）

> 本文需要读者对Spring Boot整合RabbitMQ有基本了解

## 环境介绍

|软件名称|软件版本|
|:--|:--|
|Spring Boot|2.5.3|
|Maven|3.6.3|
|RabbitMQ Server|3.9.1|
|erlang|24.0.5|

## AmqpAdmin介绍

上一篇讲了在**Spring Boot**框架中如何连一个和多个**RabbitMQ**

在例子中，我们知道如何使用`@RabbitListener`指定对应的**RabbitMQ**，有读者提出疑问，在`RabbitmqConfiguration`中创建的`Queue`、`Exchange`、`Binding`这些**Bean**又是通过什么方式与**RabbitMQ**关联起来的呢？

接下来就让我们一起学习`AmqpAdmin`这个类。

进入到**Spring Boot**定义好的配置类`RabbitAutoConfiguration`，其中有一个**Bean**创建的代码

```
		@Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic", matchIfMissing = true)
		@ConditionalOnMissingBean
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
			return new RabbitAdmin(connectionFactory);
		}
```

如果只有一个`ConnectionFactory`的时候，**Spring Boot**会帮我们创建一个默认的**RabbitAdmin**，那这个**RabbitAdmin**到底是干什么的呢？我们进入`RabbitAdmin`这个类的源码继续学习

看`public void initialize()`方法，这个方法是对`InitializingBean`方法的实现，也就是**Spring Boot**框架调用的**Bean**初始化方法，看下这个方法的关键代码，就能明白这个方法大概做了什么

```
		this.logger.debug("Initializing declarations");
		Collection<Exchange> contextExchanges = new LinkedList<Exchange>(
				this.applicationContext.getBeansOfType(Exchange.class).values());
		Collection<Queue> contextQueues = new LinkedList<Queue>(
				this.applicationContext.getBeansOfType(Queue.class).values());
		Collection<Binding> contextBindings = new LinkedList<Binding>(
				this.applicationContext.getBeansOfType(Binding.class).values());
		Collection<DeclarableCustomizer> customizers =
				this.applicationContext.getBeansOfType(DeclarableCustomizer.class).values();

		processDeclarables(contextExchanges, contextQueues, contextBindings);

		final Collection<Exchange> exchanges = filterDeclarables(contextExchanges, customizers);
		final Collection<Queue> queues = filterDeclarables(contextQueues, customizers);
		final Collection<Binding> bindings = filterDeclarables(contextBindings, customizers);
```

这段代码的意思是，获取所有`Exchange`、`Queue`、`Binding`的**Bean**，然后经过某种条件过滤，再对这些队列、交换机、绑定关系进行创建。那最为关键的过滤规则是怎样的呢？

### DeclarableCustomizer介绍

一种过滤规则是根据`DeclarableCustomizer`这个接口去过滤，顾名思义，开发人员可以自己实现这个接口，然后根据自己的业务规则进行过滤，这个接口就是一个函数式接口，有兴趣的朋友可以自己去实现。

### AmqpAdmin介绍

这次重点要讲的另外一种过滤规则，进入`filterDeclarables()`继续一探究竟

```
	private <T extends Declarable> Collection<T> filterDeclarables(Collection<T> declarables,
			Collection<DeclarableCustomizer> customizers) {

		return declarables.stream()
				.filter(dec -> dec.shouldDeclare() && declarableByMe(dec))
				.map(dec -> {
					if (customizers.isEmpty()) {
						return dec;
					}
					AtomicReference<T> ref = new AtomicReference<>(dec);
					customizers.forEach(cust -> ref.set((T) cust.apply(ref.get())));
					return ref.get();
				})
				.collect(Collectors.toList());
	}
```

重点来了，最重要的过滤规则在这一行代码`filter(dec -> dec.shouldDeclare() && declarableByMe(dec))`

`dec.shouldDeclare()`默认值就是`true`，没啥好说的，那就继续看`declarableByMe(dec)`

```
	private <T extends Declarable> boolean declarableByMe(T dec) {
		return (dec.getDeclaringAdmins().isEmpty() && !this.explicitDeclarationsOnly) // NOSONAR boolean complexity
				|| dec.getDeclaringAdmins().contains(this)
				|| (this.beanName != null && dec.getDeclaringAdmins().contains(this.beanName));
	}
```

这段代码一共包含三个条件的判断
  - `(dec.getDeclaringAdmins().isEmpty() && !this.explicitDeclarationsOnly)`，意思是`Declarable`如果没有指定`AmqpAdmin`并且没有要求显示声明的话，那么就会由当前`AmqpAdmin`创建。`this.explicitDeclarationsOnly`这个属性默认就是`false`，所以大部分情况下，创建的`AmqpAdmin`会在**RabbitMQ**里面创建所有的队列、交换机以及绑定关系，只要这些对象以**Bean**的形式存在
  - `dec.getDeclaringAdmins().contains(this)`，意思是`Declarable`指定了当前的`AmqpAdmin`才会去**RabbitMQ**创建队列、交换机以及绑定关系
  - `(this.beanName != null && dec.getDeclaringAdmins().contains(this.beanName)`，这个条件的判断与第二个条件判断是一个意思，区别是一个判断是`AmqpAdmin`的**Bean**名称，一个判断的是`AmqpAdmin`对象
  
也就是说`Declarable`在指定`AmqpAdmin`的时候，可以指定**Bean**名称，也可以直接指定对象。

## AmqpAdmin应用示例

理论学习完了，那就来动手实践吧。

在我们自己的`RabbitmqConfiguration`配置类中新增创建两个**Bean**

```
	@Bean
	public AmqpAdmin defaultAmqpAdmin(@Qualifier("defaultConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitAdmin amqpAdmin = new RabbitAdmin(connectionFactory);
		amqpAdmin.setExplicitDeclarationsOnly(true);
		return amqpAdmin;
	}
	
	@Bean
	public AmqpAdmin secondAmqpAdmin(@Qualifier("secondConnectionFactory") ConnectionFactory connectionFactory) {
		RabbitAdmin amqpAdmin = new RabbitAdmin(connectionFactory);
//		amqpAdmin.setExplicitDeclarationsOnly(true);
		return amqpAdmin;
	}
```

我们创建了两个`AmqpAdmin`，一个要求显示声明`Declarable`对象，一个则不要求

我们继续新增创建一个**队列3**的**Bean**

```Java
	@Bean
	public Queue mqQueue3() {
		String queueName = "mq-queue-3";
		return new Queue(queueName, true, false, false, new HashMap<>());
	}
	
	@Bean
	public Exchange mqExchage3() {
		String exchangeName = "mq-exchange-3";
        return new FanoutExchange(exchangeName, true, false, new HashMap<>());
    }
	
	@Bean
	public Binding bindQueue3AndExchange3(
			@Qualifier("mqQueue3") Queue mqQueue,
			@Qualifier("mqExchage3") Exchange mqExchage) {
		
		Binding binding = BindingBuilder.bind(mqQueue).to(mqExchage).with("").noargs();
        return binding;
    }
```

继续跑单元测试可以知道，这三个`Declarable`只会在`secondAmqpAdmin`中创建，并不会在`defaultAmqpAdmin`创建，因为`defaultAmqpAdmin`只会创建指定了自己为的`Declarable`对象。如果想显示指定`Declarable`的`admin`要怎么做呢？

我们重新定义下**队列1**和**队列2**的代码

```
	@Bean
	public Queue mqQueue1(@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin, @Qualifier("secondAmqpAdmin") AmqpAdmin secondAmqpAdmin) {
		String queueName = "mq-queue-1";
		Map<String, Object> arguments = null;
		Queue queue = new Queue(queueName, true, false, false, arguments);
		queue.setAdminsThatShouldDeclare(amqpAdmin, secondAmqpAdmin);
		return queue;
	}
	
	@Bean
	public Queue mqQueue2(@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		String queueName = "mq-queue-2";
		Map<String, Object> arguments = null;
		Queue queue = new Queue(queueName, false, false, true, arguments);
		queue.setAdminsThatShouldDeclare(amqpAdmin);
		return queue;
	}

	@Bean
	public Exchange mqExchage1(@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		String exchangeName = "mq-exchange-1";
		Map<String, Object> arguments = null;
		Exchange exchange = new FanoutExchange(exchangeName, true, false, arguments);
		exchange.setAdminsThatShouldDeclare(amqpAdmin);
        return exchange;
    }
	
	@Bean
	public Exchange mqExchage2(@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		String exchangeName = "mq-exchange-2";
		Map<String, Object> arguments = null;
		Exchange exchange = new FanoutExchange(exchangeName, true, false, arguments);
		exchange.setAdminsThatShouldDeclare(amqpAdmin);
        return exchange;
    }
	
	@Bean
	public Binding bindQueue1AndExchange1(
			@Qualifier("mqQueue1") Queue mqQueue1,
			@Qualifier("mqExchage1") Exchange mqExchage1,
			@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		log.debug("bind queue[{}] -> exchage[{}]", mqQueue1.getName(), mqExchage1.getName());
		
		Binding binding = BindingBuilder.bind(mqQueue1).to(mqExchage1).with("").noargs();
		binding.setAdminsThatShouldDeclare(amqpAdmin);
        return binding;
    }
	
	@Bean
	public Binding bindQueue2AndExchange2(
			@Qualifier("mqQueue2") Queue mqQueue2,
			@Qualifier("mqExchage2") Exchange mqExchage2,
			@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		log.debug("bind queue[{}] -> exchage[{}]", mqQueue2.getName(), mqExchage2.getName());
		Binding binding = BindingBuilder.bind(mqQueue2).to(mqExchage2).with("").noargs();
		binding.setAdminsThatShouldDeclare(amqpAdmin);
        return binding;
    }
```

通过`setAdminsThatShouldDeclare`方法可以指定由谁来创建，这样就可以根据业务需要声明多个队列，然后显示指定在哪个**RabbitMQ**中创建这些队列。

## 如何创建并订阅一个动态名称的队列

上一篇讲了在**Spring Boot**框架中如何连一个和多个**RabbitMQ**

在例子中，使用的队列名称是静态的，无论是在代码里面写死一个名称，还是通过配置文件中指定队列名称，都是在程序启动前已经确定了

那如果想来点不一样的名称呢？比如队列名称带上本机IP，比如队列名称带上程序的进程ID，可以做到吗？

答案是肯定的。

如果想在`@RabbitListener`这个注解里面指定一个动态获取的名称是不可以的，会报编译错误，因为只能指定静态常量。那我们接下来就换一种方式来实现。是的，我们可以创建**Bean**的时候，指定这个队列的名称

```
	@Bean
	public Queue mqQueue2(@Qualifier("defaultAmqpAdmin") AmqpAdmin amqpAdmin) {
		String queueName = "mq-queue-2-" + getSiteLocalAddress() + "-" + getPid();
		Map<String, Object> arguments = null;
		Queue queue = new Queue(queueName, false, false, true, arguments);
		queue.setAdminsThatShouldDeclare(amqpAdmin);
		return queue;
	}
```

`getSiteLocalAddress()`的代码如下

```
	public static String getSiteLocalAddress() {
		try {
			Enumeration<NetworkInterface> netInterfaces = NetworkInterface.getNetworkInterfaces();
			while (netInterfaces.hasMoreElements()) {
				Enumeration<InetAddress> inetAddress = netInterfaces.nextElement().getInetAddresses();
				while (inetAddress.hasMoreElements()) {
					InetAddress ip = inetAddress.nextElement();
					String hostAddress = ip.getHostAddress();
					if (!ip.isLoopbackAddress() && hostAddress.length() <= 16) {
						return hostAddress;
					}
				}
			}
		} catch (Exception e) {
			log.error("", e);
		}

		return "";
	}
```

`getPid()`代码如下

```
	public static String getPid() {
		try {
			String processName = ManagementFactory.getRuntimeMXBean().getName();
			String processID = processName.substring(0, processName.indexOf('@'));
			return processID;
		} catch (Exception e) {
			log.error("", e);
		}
		return "";
	}
```

看看**RabbitMQ**的列表

![带IP和进程号的队列名称](https://img-blog.csdnimg.cn/495eae1a5920468ca5dad2b52596b70f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyaWVuZGx5dGt5ag==,size_16,color_FFFFFF,t_70#pic_center "带IP和进程号的队列名称")

