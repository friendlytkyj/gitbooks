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

```
@Bean(name = "rabbitListenerContainerFactory")
	SimpleRabbitListenerContainerFactory defaultRabbitListenerContainerFactory(
			SimpleRabbitListenerContainerFactoryConfigurer configurer,
			@Qualifier("defaultConnectionFactory") ConnectionFactory connectionFactory) {
		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		return factory;
	}
```

```
DeclarableCustomizer

	private <T extends Declarable> boolean declarableByMe(T dec) {
		return (dec.getDeclaringAdmins().isEmpty() && !this.explicitDeclarationsOnly) // NOSONAR boolean complexity
				|| dec.getDeclaringAdmins().contains(this)
				|| (this.beanName != null && dec.getDeclaringAdmins().contains(this.beanName));
	}
```


```
	/**
	 * Declares all the exchanges, queues and bindings in the enclosing application context, if any. It should be safe
	 * (but unnecessary) to call this method more than once.
	 */
	@Override // NOSONAR complexity
	public void initialize() {

		if (this.applicationContext == null) {
			this.logger.debug("no ApplicationContext has been set, cannot auto-declare Exchanges, Queues, and Bindings");
			return;
		}

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

		for (Exchange exchange : exchanges) {
			if ((!exchange.isDurable() || exchange.isAutoDelete())  && this.logger.isInfoEnabled()) {
				this.logger.info("Auto-declaring a non-durable or auto-delete Exchange ("
						+ exchange.getName()
						+ ") durable:" + exchange.isDurable() + ", auto-delete:" + exchange.isAutoDelete() + ". "
						+ "It will be deleted by the broker if it shuts down, and can be redeclared by closing and "
						+ "reopening the connection.");
			}
		}

		for (Queue queue : queues) {
			if ((!queue.isDurable() || queue.isAutoDelete() || queue.isExclusive()) && this.logger.isInfoEnabled()) {
				this.logger.info("Auto-declaring a non-durable, auto-delete, or exclusive Queue ("
						+ queue.getName()
						+ ") durable:" + queue.isDurable() + ", auto-delete:" + queue.isAutoDelete() + ", exclusive:"
						+ queue.isExclusive() + ". "
						+ "It will be redeclared if the broker stops and is restarted while the connection factory is "
						+ "alive, but all messages will be lost.");
			}
		}

		if (exchanges.size() == 0 && queues.size() == 0 && bindings.size() == 0) {
			this.logger.debug("Nothing to declare");
			return;
		}
		this.rabbitTemplate.execute(channel -> {
			declareExchanges(channel, exchanges.toArray(new Exchange[exchanges.size()]));
			declareQueues(channel, queues.toArray(new Queue[queues.size()]));
			declareBindings(channel, bindings.toArray(new Binding[bindings.size()]));
			return null;
		});
		this.logger.debug("Declarations finished");

	}
```

```
	/**
	 * Remove any instances that should not be declared by this admin.
	 * @param declarables the collection of {@link Declarable}s.
	 * @param customizers a collection if {@link DeclarableCustomizer} beans.
	 * @param <T> the declarable type.
	 * @return a new collection containing {@link Declarable}s that should be declared by this
	 * admin.
	 */
	@SuppressWarnings("unchecked")
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
