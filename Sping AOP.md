# Sping AOP使用

## 概念

1. 通知(Advice)

   在AOP术语中，切面要完成的工作被称为通知，通知定义了切面是什么以及何时使用。

2. 连接点(Join point)

   连接点是在应用执行过程中能够插入切面的一个点，这个点可以是调用方法时、抛出异常时、修改某个字段时。

3. 切点(Pointcut)

   切点是为了缩小切面所通知的连接点的范围，即切面在何处执行。我们通常使用明确的类和方法名称，或者利用正则表达式定义所匹配的类和方法名称来指定切点。

4. 切面(Aspect)

   切面是通知和切点的结合。通知和切点共同定义了切面的全部内容：它是什么，在何时和何处完成其功能。

5. 引入(Introduction)

   引入允许我们在不修改现有类的基础上，向现有类添加新方法或属性。

6. 织入(Weaving)

   织入是把切面应用到目标对象并创建新的代理对象的过程。

## 使用

1. ### 新增一个配置类

   加上注解：

   ```java
   @Configuration //标识该文件是配置文件
   @EnableAspectJAutoProxy //启用AOP
   ```

2. ### 创建一个切面组建

   加入注解：

   ```java
   @Component
   @Aspect
   @Before("execution(* com.example.demo.controller.Test.*(..))")
   ```

   
   - 前置通知(Before)：在目标方法被调用之前调用通知功能
   - 后置通知(After)：在目标方法完成之后调用通知，此时不关心方法的输出结果是什么
   - 返回通知(After-returning)：在目标方法成功执行之后调用通知
   - 异常通知(After-throwing)：在目标方法抛出异常后调用通知
   - 环绕通知(Around)：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为
   - 可复用的切点(Pointcut):定义一个可复用的切点
   
   
   
   注解value用法
   
   - execution(比配方法签名)、within(匹配类型签名)、bean(匹配Bean名称)、args(匹配参数)、@annotation(匹配由指定注解所标注的方法)
   - 方法执行时触发(execution)
   - 表明我们不关心方法返回值的类型，即可以是任意类型(*)
   - 使用全限定类名和方法名指定要添加前置通知的方法(com.example.demo.controller.Test)
   - 方法的参数列表使用(..)，表明我们不关心方法的入参是什么，即可以是任意类型



# 支持

[Spring入门(十)：Spring AOP使用讲解 (juejin.cn)](https://juejin.cn/post/6844903925112373262)

[彻底征服 Spring AOP 之 理论篇 - SegmentFault 思否](https://segmentfault.com/a/1190000007469968)

