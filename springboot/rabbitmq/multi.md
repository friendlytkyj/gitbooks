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

## 配置多个MongoDB数据源

### 自定义数据源配置项

`spring.data.mongodb`配置是**Spring Boot**框架自己能够识别的配置，所以可以为我们创建`MongoTemplate`对象供我们使用，如果要实现多个MongoDB数据源的配置，就需要我们自己读取配置自己创建MongoTemplate对象。

比如，我们想配置2个MongoDB数据源，分别取名叫xxx1和xxx2，并定义好如下配置项

```
spring.data.mongodb.xxx1.uri: mongodb://localhost:27017/Teach1
spring.data.mongodb.xxx2.uri: mongodb://localhost:27017/Teach2
```

### 如何创建自定义MongoTemplate对象

想要实现自定义多个数据源，就需要做到像**Spring Boot**框架那样构造一个`MongoTemplate`对象，接下来我们跟踪源码学习如何自己手工创建`MongoTemplate`

#### 查看MongoTemplate.java源码

进入`MongoTemplate.java`源代码，查看构造函数

```
public MongoTemplate(MongoClient mongoClient, String databaseName)

public MongoTemplate(MongoDatabaseFactory mongoDbFactory)

public MongoTemplate(MongoDatabaseFactory mongoDbFactory, @Nullable MongoConverter mongoConverter)
```

一共有三个可用的构造函数，从构造函数可以看出，如果想构造一个`MongoTemplate`对象，就需要先创建一个`MongoClient`对象或者`MongoDatabaseFactory`对象

#### 追踪MongoClient对象如何创建

在`MongoTemplate.java`中可以看到**SimpleMongoClientDatabaseFactory**这个对象的创建，创建这个对象需要传入一个MongoClient对象，利用IDE工具查看**SimpleMongoClientDatabaseFactory**构造函数调用的地方

![调用SimpleMongoClientDatabaseFactory构造函数的地方](https://img-blog.csdnimg.cn/d4235d63a7234277b5dfa62df22c0493.png#pic_center "调用SimpleMongoClientDatabaseFactory构造函数的地方")

进入`mongoDbFactory()`

```
	@Bean
	public MongoDatabaseFactory mongoDbFactory() {
		return new SimpleMongoClientDatabaseFactory(mongoClient(), getDatabaseName());
	}
```

再跟踪进入`mongoClient()`

```
	public MongoClient mongoClient() {
		return createMongoClient(mongoClientSettings());
	}
```

再跟踪进入`createMongoClient()`

```
	protected MongoClient createMongoClient(MongoClientSettings settings) {
		return MongoClients.create(settings, SpringDataMongoDB.driverInformation());
	}
```

从这里可以看到，原来有一个工具类`MongoClients`帮助创建`MongoClient`对象，这个时候我们进一步查看这个工具类其它`create`方法

![MongoClients.create](https://img-blog.csdnimg.cn/a07e54d24f65402cbba0288a5d2bff8e.png#pic_center "MongoClients.create")

其中有一个最简单的工具方法，`String`类型的`connectionString`参数,这个`connectionString`其实就是连接**MongoDB**的**URI**

> 源码看到这里，我们可以知道，如果要创建`MongoTemplate`对象，无论是传`MongoClient`还是`MongoDatabaseFactory`对象，最终都是需要一个`connectionString`，也就是**uri**配置，那我们接下来就可以读取配置并自己构造`MongoTemplate`对象了。

### 读取自定义数据源配置项

上面这个配置项，**Spring Boot**框架是不认识的，所以需要我们自己去完成对配置的读取，代码参考如下

```
package com.etomy.teach.mongodb.properties;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;

@Configuration
public class MgoConfiguration2 {
	
	@Bean
	public MongoTemplate xxx1MongoTemplate(@Value("${spring.data.mongodb.xxx1.uri}") String xxx1Url) {
		return new MongoTemplate(new SimpleMongoClientDatabaseFactory(xxx1Url));
	}
	
	@Bean
	public MongoTemplate xxx2MongoTemplate(@Value("${spring.data.mongodb.xxx2.uri}") String xxx2Url) {
		return new MongoTemplate(new SimpleMongoClientDatabaseFactory(xxx2Url));
	}
}
```

### 单元测试

上面这个类完成对xxx1和xxx2两个mongodb数据源配置的读取，并且构造两个`MongoTemplate` Bean

`MongoTemplate`有了，接下来写单元测试

```
package com.etomy.teach.mongodb.properties;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.test.context.TestPropertySource;

import com.etomy.teach.mongodb.SpringbootTestBase;

@TestPropertySource(properties = {
		"spring.data.mongodb.xxx1.uri: mongodb://localhost:27017/Teach1",
		"spring.data.mongodb.xxx2.uri: mongodb://localhost:27017/Teach2"
})
public class MgoConfiguration2Test extends SpringbootTestBase {

	@Qualifier("xxx1MongoTemplate")
	private @Autowired MongoTemplate xxx1MongoTemplate;
	
	@Qualifier("xxx2MongoTemplate")
	private @Autowired MongoTemplate xxx2MongoTemplate;
	
	@Test
	public void testMultiMongodb() {
		xxx1MongoTemplate.insert(buildMgoMessage());
		xxx2MongoTemplate.insert(buildMgoMessage());
	}
}
```

### 核对测试结果

运行单元测试后，从mongodb核对数据情况

- xxx1数据源

![xxx1数据源](https://img-blog.csdnimg.cn/0f71ac4abf3647198aee3243a9cc7b2d.png#pic_center "xxx1数据源")

- xxx2数据源

![xxx2数据源](https://img-blog.csdnimg.cn/e08c9d8585634d88a85cb48d75675d48.png#pic_center "xxx2数据源")

### 单元测试说明

- 在**MgoConfiguration2**中创建了两个**Bean**，**Bean**的名字分别是**xxx1MongoTemplate**和**xxx2MongoTemplate**
- 在**MgoConfiguration2Test**单元测试中，通过**@Qualifier**指定**Bean**获取**MgoConfiguration2**中定义的两个**Bean**
- 最后通过使用两个数据源的**MongoTemplate**操作数据库

### 插入的数据去掉_class字段

直接上代码

```
	public void removeClassField(MongoTemplate mongoTemplate) {
		MongoConverter converter = mongoTemplate.getConverter();
	    if (converter.getTypeMapper().isTypeKey(TYPE_KEY)) {
	      ((MappingMongoConverter) converter).setTypeMapper(new DefaultMongoTypeMapper(null));
	    }
	}
```