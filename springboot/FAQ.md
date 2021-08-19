# @AutoConfigureBefore和@AutoConfigureAfter不生效

当使用这2个标签不生效的时候，是因为`@Autowired`会直接去找依赖的**Bean**

解决方法: 增加`@Lazy`，与`@Autowired`一起使用，**Spring Boot**注入的时候会等所有的Bean都扫描完再延迟注入

# @RestController和@Controller

- 如果返回JSON等自定义数据，使用`@RestController`
- 如果返回HTML等页面，使用`@Controller`


