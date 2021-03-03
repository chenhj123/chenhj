# Spring中注解

1. @Transactional

   Spring 事务管理分为编码式和声明式的两种方式

   编程式事务指的是通过编码方式实现事务；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。

   两种声明式事务方式，1、在配置文件（xml）中做相关的事务规则声明，2、基于 @Transactional 注解的方式

   注解可以添加在方法上，也可以添加到类级别上

   注解属性信息：

   - name（当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器）
   - propagation（事务的传播行为，默认值为 REQUIRED）
   - isolation（事务的隔离度，默认值采用 DEFAULT。）
   - timeout（事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务）
   - read-only（指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true）
   - rollback-for（用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔）
   - no-rollback- for（抛出 no-rollback-for 指定的异常类型，不回滚事务）

   注意：

   - 正确的设置 @Transactional 的 propagation 属性
   - 正确的设置 @Transactional 的 rollbackFor 属性
   - @Transactional 只能应用到 public 方法才有效
   - 避免 Spring 的 AOP 的自调用问题（使用 AspectJ 取代 Spring AOP 代理可解决）

2. @Controller和@RestController区别

   - 如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，或者html，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return里的内容
   - 如果需要返回到指定页面，则需要用@Controller配合视图解析器InternalResourceViewResolver才行，假设要返回 Json，XML 或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBode注解。

# 支持

[透彻的掌握 Spring 中 @transactional 的使用 – IBM Developer](https://developer.ibm.com/zh/languages/java/articles/j-master-spring-transactional-use/)