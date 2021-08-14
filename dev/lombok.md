# 常用注解

- @Getter/@Setter: 作用类上，生成所有成员变量的getter/setter方法；作用于成员变量上，生成该成员变量的getter/setter方法。可以设定访问权限及是否懒加载等。

- @ToString：作用于类，覆盖默认的toString()方法，可以通过of属性限定显示某些字段，通过exclude属性排除某些字段。

- @EqualsAndHashCode：作用于类，覆盖默认的equals和hashCode

- @NonNull：主要作用于成员变量和参数中，标识不能为空，否则抛出空指针异常。

- @NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor：作用于类上，用于生成构造函数。有staticName、access等属性。
  staticName属性一旦设定，将采用静态方法的方式生成实例，access属性可以限定访问权限。

- @NoArgsConstructor：生成无参构造器

- @RequiredArgsConstructor：生成包含final和@NonNull注解的成员变量的构造器；

- @AllArgsConstructor：生成全参构造器

- @Data：作用于类上，是以下注解的集合：@ToString @EqualsAndHashCode @Getter @Setter @RequiredArgsConstructor

- @Builder：作用于类上，将类转变为建造者模式

- @Log：作用于类上，生成日志变量。针对不同的日志实现产品，有不同的注解：

- @Cleanup：自动关闭资源，针对实现了java.io.Closeable接口的对象有效，如：典型的IO流对象

- @SneakyThrows：可以对受检异常进行捕捉并抛出

- @FieldNameConstants 为每个属性生成一个字段名的字符串常量

- @Accessors(chain = true) 链式调用Setter