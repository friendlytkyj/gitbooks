# Spring Boot整合Junit 5

> 要求读者对**Spring Boot**有基本的了解，本文不再对**Spring Boot**做基本介绍。

本文主要介绍**Spring Boot**与**Junit**整合，实现单元测试。

## 环境介绍

|软件名称|软件版本|
|:--|:--|
|Spring Boot|2.5.3|
|Maven|3.6.3|

## 搭建一个maven工程

修改`pom.xml`，指定父级依赖

```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.5.3</version>
</parent>
```

## 编写一个单元测试的父类

```
/**
 * 单元测试可以继承此类。
 * 
 * @author Etomy
 */
@SpringBootTest(classes = Application.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public abstract class AbstractSpringbootTestBase extends Assertions {
	
}
```

## 开始写单元测试

```
package com.etomy.teach.junit;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class AnnotationTest extends AbstractSpringbootTestBase {
	
	private static final String LOGGER_TEST = "==== {} ====";

	// @BeforeAll 类似于JUnit 4的@BeforeAll,表示使用了该注解的方法应该在当前类中所有使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之前执行，必须为static
	@BeforeAll
	public static void beforeAll() {
		log.debug(LOGGER_TEST, 1);
	}
	
	// @BeforeEach 类似于JUnit 4的@Before,表示使用了该注解的方法应该在当前类中每一个使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之前执行
	@BeforeEach
	public void beforeEach() {
		log.debug(LOGGER_TEST, 2);
	}
	
	// @AfterEach 类似于JUnit 4的@After,表示使用了该注解的方法应该在当前类中每一个使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之后执行。
	@AfterEach 
	public void afterEach () {
		log.debug(LOGGER_TEST, 3);
	}
		
	// @AfterAll 类似于JUnit 4的@AfterClass,表示使用了该注解的方法应该在当前类中所有使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之后执行,必须为static
	@AfterAll 
	public static void afterAll () {
		log.debug(LOGGER_TEST, 4);
	}

	// @DisplayName 为测试类或测试方法声明一个自定义的显示名称。
	@DisplayName("ttttttttt")
	// @Test 表示该方法是一个测试方法。
	@Test
	public void test() {
		log.debug(LOGGER_TEST, "hello world");
	}
}
```

## 常用注解

### @BeforeAll

类似于JUnit 4的@BeforeAll,表示使用了该注解的方法应该在当前类中所有使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之前执行，必须为static

### @BeforeEach

类似于JUnit 4的@Before,表示使用了该注解的方法应该在当前类中每一个使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之前执行

### @AfterEach

类似于JUnit 4的@After,表示使用了该注解的方法应该在当前类中每一个使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之后执行。

### @AfterAll

类似于JUnit 4的@AfterClass,表示使用了该注解的方法应该在当前类中所有使用了@Test、@RepeatedTest、@ParameterizedTest或者@TestFactory注解的方法之后执行,必须为static

### @DisplayName

为测试类或测试方法声明一个自定义的显示名称。

![@DisplayName注解使用效果图](https://img-blog.csdnimg.cn/82c822b17e2d4fff9dbce30e68de4b1a.png#pic_left "@DisplayName注解使用效果图")




