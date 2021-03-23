# 面试题

1. ### Java基本数据类型有哪些

   byte：-128（-2^7）到 127（2^7-1）

   short：-32768（-2^15）到 32767（2^15 - 1）

   int：-2,147,483,648（-2^31）到 2,147,483,647（2^31 - 1）

   long：-9,223,372,036,854,775,808（-2^63）到 9,223,372,036,854,775,807（2^63 -1）

   float：32位单精度

   double：64位双精度

   char：char类型是一个单一的 16 位 Unicode 字符，\u0000 到 \uffff

   boolean：true 或 false

2. ### String与StringBuffer有什么区别

   - String类的内容一旦声明之后是不可改变的，改变的只是其内存的指向，而StringBuffer的对象内容是可以改变的。String对象不可修改指的是对象本身不可修改，而不是引用不可修改

     举个栗子：

     假如有 String a="你好"; 意思是声明一个String类型的引用变量a,在内存中创建一个String对象（值为"你好"），然后把这个对象的引用赋值给变量a。

     a = "hello"; 这行代码执行的效果是在内存中另外创建了一个String对象（值为"hello"），然后把这个新对象的引用赋值给变量a，而不是把原来的内存中的那个“你好”的String对象值变为“hello”。

   - 对于StringBuffer，不能像String那样直接通过赋值的方式完成对象实例化，必须通过构造方法的方式完成

     StringBuffer sb=new StringBuffer();

   - StringBuffer类在进行字符串处理时，不生成新的对象，在内存使用上要优于String类。所以在实际使用时，如果经常需要对一个字符串进行修改，例如插入，删除等操作，使用StringBuffer要更加适合一些

   - **StringBuffer** 线程安全、**StringBuilder** 非线程安全

     两者的区分在于：**StringBuffer** 中改变字符串的基本上都加上了 synchronized 且 toString 方法中会复制一份数组存放在缓存中

3. ### transient的作用及使用方法

   我们都知道一个对象只要实现了Serilizable接口，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。
   
   然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。
   
   总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。
   
4. ### 单例模式

   https://github.com/chenhj123/chenhj/blob/main/singleton.md

5. ### mybatis的原理

   https://juejin.cn/post/6844903585298251789

6. ### spring aop是什么

   https://segmentfault.com/a/1190000007469968

7. ### 数据库优化的思路

   https://blog.csdn.net/AnneQiQi/article/details/51447577

8. ### servlet的生命周期

   https://blog.csdn.net/hanziang1996/article/details/78965791
   
9. ### 地址

   ```java
   public class equalDemo {
       public static void main(String[] args) {
           //先当与新建两个对象，地址不一致，返回false
           Integer value1 = new Integer(1);
           Integer value2 = new Integer(1);
           System.out.println(value1 == value2);
   		//相当于两个int，返回true
           int value3 = 1;
           int value4 = 1;
           System.out.println(value3 == value4);
   		//局部变量不影响外部变量，所以s1还是zhangsan
           String s1 = "zhangsan";
           exchange(s1);
           System.out.println(s1);
   		//s1地址，指向"list"，所以s1为lisi
           s1 = "lisi";
           System.out.println(s1);
   		//两个地址不一致，所以为false
           String str1 = new String("aa");
           String str2 = new String("aa");
           System.out.println(str1 == str2);
           //int为基本类型，判断是否相等就可以使用==
           //但是Integer cache中已有-128到127,大于这个范围就会产生新的new
   		Integer c = 3;
           Integer d = 3;
           Integer e = 321;
           Integer f = 321;
           System.out.println(c == d);
        System.out.println(e == f);
       }
   
       private static void exchange(String s1){
           s1 = "lisi";
       }
   }
   ```
   
10. Spring 容器的加载过程

    

11. Spring IOC 怎么存储对象的

    concurrentHashMap

    首先Spring IOC容器是ApplicationContext，ApplicationContext又有上下级，同时委托给beanFactory管理对象。管理对象分三种：自定义bean，内建依赖，收工托管单例。Scope又分为单例、原型等（实例，beandefinition）,还有类型，名称，别名，加载顺序，以及依赖，都会保存

12. Spring 怎么处理循环依赖

    - Spring是通过递归的方式获取目标bean及其所依赖的bean的；
    - Spring实例化一个bean的时候，是分两步进行的，首先实例化目标bean，然后为其注入属性。

    结合这两点，也就是说，Spring在实例化一个bean的时候，是首先递归的实例化其所依赖的所有bean，直到某个bean没有依赖其他bean，此时就会将该实例返回，然后反递归的将获取到的bean设置为各个上层bean的属性的。

13. HashMap 为什么线程不安全

14. volitile关键字的作用

    - 保证内存可见性
    - 禁止指令重排

15. 嗅探机制和缓存行是怎么回事

    MESI缓存一致性协议

    CPU缓存的最小单位：缓存行（Cache line），有可能32字节、64字节、128字节，根据CPU来定；

    https://www.cnblogs.com/yufeng218/p/12130406.html

16. 线程池的执行流程

17. 一个未知大小巨量的数字数据求和，用什么线程池好

18. 并发工具用过什么应用场景

19. 举一个多线程开发的例子

20. 初始化对象，各个值在 JVM 中怎么分配

21. 什么情况下对象会直接到栈中

22. SQL 优化

23. 对微服务的理解，罗列一下相关技术框架

24. HashMap数据结构

    HashMap的底层主要是基于数组和链表来实现的，它之所以有相当快的查询速度主要是因为它是通过计算散列码来决定存储的位置。HashMap中主要是通过key的hashCode来计算hash值的，只要hashCode相同，计算出来的hash值就一样。如果存储的对象对多了，就有可能不同的对象所算出来的hash值是相同的，这就出现了所谓的hash冲突。学过数据结构的同学都知道，解决hash冲突的方法有很多，HashMap底层是通过链表来解决hash冲突的。

25. Java1.7和Java1.8有什么不同

    - 新增stream
    - Hashmap优化
    - 移除永元代，变成元空间
    - Lamda表达式
    - 并发：LongAdder
    - CompletableFuture
    - StampedLock
    - concurrentHashMap

26. concurrentHashmap怎么实现线程安全

    - 使用volatile保证当Node中的值变化时对于其他线程是可见的
    - 使用table数组的头节点作为synchronized的锁来保证写操作的安全
    - 当头节点为null时，使用CAS操作来保证数据能正确的写入

27. synchronized 锁和 ReentrantLock 区别，为什么1.8改成 synchronized ，优化了什么

    aaaa

28. 怎么理解偏向锁、轻量级锁、重量级锁

    偏向锁：无实际竞争，且将来只有一个申请锁的线程会使用锁。

    轻量级锁：无实际竞争，多个线程交替使用锁；允许短时间的锁竞争。

    重量级锁：有实际竞争，且锁竞争时间长

29. SpringBean 的生命周期

    实例化->属性赋值->初始化->销毁

30. Springboot 和  Spring 有哪些区别

    - 约定大于配置；
    - 创建独立运行的 Spring 应用；
    - 直接嵌入 Tomcat 或 Jetty，Undertow，无需部署 WAR 包；
    - 提供限定性的 starter 依赖简化配置（就是脚手架）；
    - 在必要时自动化配置 Spring 和其他三方依赖库
    - 提供生产 production-ready 特性，例如指标度量，健康检查，外部配置等
    - 完全零代码生产和不需要 XML 配置

31. mq 和 缓存作为中间件有什么区别，分别作用是什么

    缓存：1.请求去重；2.恢复数据；3.预读取；4.临时存储

    mq：1.超时订单；2.解耦；3.流量消峰；4.消息分发；5.异步消息