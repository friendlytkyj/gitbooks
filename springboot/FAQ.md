# @AutoConfigureBefore和@AutoConfigureAfter不生效

当使用这2个标签不生效的时候，是因为`@Autowired`会直接去找依赖的**Bean**

解决方法: 增加`@Lazy`，与`@Autowired`一起使用，**Spring Boot**注入的时候会等所有的Bean都扫描完再延迟注入

# @RestController和@Controller

- 如果返回JSON等自定义数据，使用`@RestController`
- 如果返回HTML等页面，使用`@Controller`

# springboot中的@profile-active@不生效

- 可以使用@..@的方式在application.yml或者application.properties文件中引用pom.xml文件中的属性变量（<properties>标签下的子标签，包括激活的profile下的属性）
- 可以通过一个maven插件修改默认的引用方式@..@，比如要改成如下这样

```dtd
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <configuration>
        <delimiters>
            <delimiter>$</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin>
```

- 生效有两种方式

第一种是使用的是spring-boot官方parent，这样parent里已经设置好了资源目录，如：

```dtd
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>
```

第二种就是未使用官方parent，需要手动设置一下资源目录，filtering=true是让资源文件解析变量

```dtd
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```