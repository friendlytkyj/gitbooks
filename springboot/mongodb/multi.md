# Spring Boot怎么实现配置多个Mongodb数据源

> 本文需要读者对Spring Boot整合Mongodb有基本了解

# 目录

- [环境介绍](#环境介绍)
- [配置单个Mongodb数据源](#配置单个Mongodb数据源)
- [配置多个MongoDB数据源](#配置多个MongoDB数据源)

## 环境介绍

|软件名称|软件版本|
|:--|:--|
|Spring Boot|2.5.3|
|Maven|3.6.3|
|MongoDB |4.2.5|

## 配置单个Mongodb数据源

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