@[TOC](目录)

# Spring Boot怎么实现配置任意数量Mongodb数据源

> 本文会教大家如何按需配置**MongoDB**数据源，代码写完后，支持配置任意数量的**MongDB**数据源，只需要根据运维需要手动增加数据源的配置，不需要再修改代码。

> 如果大家只想了解配置1个或有限数量的多个数据源，可以参考另一篇文章: [【Spring Boot怎么实现配置多个Mongodb数据源】][SpringBoot+Mongodb], 里面有简单的示例代码帮助大家理解。

[SpringBoot+Mongodb]:https://blog.csdn.net/friendlytkyj/article/details/119465801

## 环境介绍

|软件名称|软件版本|
|:--|:--|
|Spring Boot|2.5.3|
|Maven|3.6.3|
|MongoDB |4.2.5|

## 多个数据源如何在配置文件中配置

在Java中，能动态存放未知数量元素的容器叫集合，我们使用**List**来存放**MongDB**数据源配置，**List**在配置文件中是以数组的方式进行配置

```
spring.data.mongodb.xxx[0].uri: mongodb://localhost:27017/Teach"
spring.data.mongodb.xxx[1].uri: mongodb://localhost:27017/Teach1"
                 ...
                 ...
spring.data.mongodb.xxx[n].uri: mongodb://localhost:27017/Teachn"
```

## 动态注册Bean

> [【Spring Boot怎么实现配置多个Mongodb数据源】][SpringBoot+Mongodb]在这篇文章中介绍了`MongoTemplate`对象的创建，但使用的是**@Bean**注解手工创建**Bean**，这种方式只能创建预先定义好的数据源，无法解决动态创建**Bean**的目的。

### BeanDefinitionRegistry介绍

通过**Spring Boot**源码可以知道，`BeanDefinitionRegistry`这个类可以通过`registerBeanDefinition()`方法注册一个`BeanDefinition`对象

```
beanDefinitionRegistry.registerBeanDefinition("xxx"+i, mongoTemplateDefinition);
```

### 如何构造BeanDefinition

借助`BeanDefinitionBuilder`这个类`genericBeanDefinition()`这个方法可以帮助我们比较容易的创建一个**BeanDefinition**

```
BeanDefinition mongoTemplateDefinition = BeanDefinitionBuilder
		.genericBeanDefinition(MongoTemplate.class, ()->mongoTemplate)
		.getBeanDefinition();
```

### 完整代码

到了这一步，可以看到，只要能构造`MongoTemplate`对象，就能把整个动态注册**Bean**的逻辑串起来了，接下来看一下通过读取自定义数据源配置动态注册Bean的完整代码

```
package com.etomy.teach.mongodb.properties;

import java.util.List;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.mongo.MongoProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;
import org.springframework.util.CollectionUtils;

@ConditionalOnProperty(name = "spring.data.mongodb.xxx[0].uri")
@Configuration
@ConfigurationProperties(prefix="spring.data.mongodb")
public class MgoConfigurationAny implements InitializingBean {
	
	private @Autowired ConfigurableApplicationContext context;
	
	private List<MongoProperties> xxx;

	public void setXxx(List<MongoProperties> xxx) {
		this.xxx = xxx;
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		if(CollectionUtils.isEmpty(xxx)) {
			return;
		}
		
		BeanDefinitionRegistry beanDefinitionRegistry = (BeanDefinitionRegistry) context;

		for(int i=0; i<xxx.size(); ++i) {
			MongoTemplate mongoTemplate = new MongoTemplate(new SimpleMongoClientDatabaseFactory(xxx.get(i).getUri()));
			BeanDefinition mongoTemplateDefinition = BeanDefinitionBuilder
					.genericBeanDefinition(MongoTemplate.class, ()->mongoTemplate)
					.getBeanDefinition();

			String beanName = "xxx"+i+"MongoTemplate";
			beanDefinitionRegistry.registerBeanDefinition(beanName, mongoTemplateDefinition);
		}
	}
}
```

## 编写单元测试代码

```
package com.etomy.teach.mongodb.properties;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.test.context.TestPropertySource;

import com.etomy.teach.mongodb.SpringbootTestBase;

@TestPropertySource(properties = {
		"spring.data.mongodb.xxx[0].uri: mongodb://localhost:27017/Teach1",
		"spring.data.mongodb.xxx[1].uri: mongodb://localhost:27017/Teach2"
})
public class MgoConfigurationAnyTest extends SpringbootTestBase {

	@Qualifier("xxx0MongoTemplate")
	private @Autowired MongoTemplate xxx1MongoTemplate;
	
	@Qualifier("xxx1MongoTemplate")
	private @Autowired MongoTemplate xxx2MongoTemplate;
	
	@Test
	public void testAnyMongodb() {
		xxx1MongoTemplate.insert(buildMgoMessage());
		xxx2MongoTemplate.insert(buildMgoMessage());
	}
}
```

## 核对数据

分别连上配置的数据源，然后查看数据是否成功插入

```
> db.DemoMessage.find().sort({sendTime:-1}).limit(1);
{ "_id" : "eb6a5b12-2b36-4f14-a50f-99275141e651", "from" : "张三", "to" : "李四", "content" : "hello, nice to meet you!", "sendTime" : NumberLong("1628315176800"), "_class" : "com.etomy.teach.mongodb.entity.MgoMessage" }
>
```

数据插入成功，测试通过。以上就是动态数据源配置的实现。

