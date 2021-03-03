# Spring中注解

1. @Transactional

   Spring 事务管理分为编码式和声明式的两种方式

   编程式事务指的是通过编码方式实现事务；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。

   两种声明式事务方式，1、在配置文件（xml）中做相关的事务规则声明，2、基于 @Transactional 注解的方式

2. @Controller和@RestController区别

   - 如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，或者html，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return里的内容
   - 如果需要返回到指定页面，则需要用@Controller配合视图解析器InternalResourceViewResolver才行，假设要返回 Json，XML 或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBode注解。

# 支持

[透彻的掌握 Spring 中 @transactional 的使用 – IBM Developer](https://developer.ibm.com/zh/languages/java/articles/j-master-spring-transactional-use/)