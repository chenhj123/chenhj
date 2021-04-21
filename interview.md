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
    
32. 常见的线程安全的 List 和 Map 有哪些？

    - Vector 和 SynchronizedList，都是直接加 Synchronized

    - CopyOnWriteArrayList，复制副本，副本上进行操作，再指向原副本

    - hashtable，直接加 Synchronized

    - synchronizedMap，每次操作hashmap都需要先获取这个对象锁

    - ConcurrentHashMap，粒度锁

33. 怎么样让双亲委派机制失效？

    - 自定义类加载器，重写loadClass方法

34. 对象的初始化过程

    - 找出.class文件，加载过程中发现有基类(通过 extends 关键字)，则先加载基类
    - 执行对象，先给对象分配存储空间，到构造函数，创建默认的属性
    - 执行构造函数余下的部分
    - 对默认属性和方法分别进行赋值和初始化

35. 接口如何限流 何时需要限流 

    单点应用，对应用入口进行限流，能达到效果

    分布式应用，可加 Nacos 进行限流

    限流算法：固定窗口计数器、滑动窗口计数器、漏桶、令牌桶

    出现大量访问时需要限流，比如爬虫，热点事件，恶意攻击等等

    [接口限流实践_清茶的博客-CSDN博客_接口限流](https://blog.csdn.net/m0_38001814/article/details/111099678)

    

36. lambda表示引用外部变量为何需要final修饰 

    说实话，我看不太懂

    [为什么lambda表达式要用final](https://blog.csdn.net/lwwgtm/article/details/60478936)

37. ThreadLocal为什么容易造成内存泄漏 如何避免 

    ThreadLocalMap 使用 ThreadLocal 的弱引用作为 key，如果一个 ThreadLocal 没有外部强引用来引用它，那么系统 GC 的时候，这个 ThreadLocal 势必会被回收，这样一来，ThreadLocalMap 中就会出现 key 为 null 的 Entry，就没有办法访问这些 key 为 null 的 Entry 的 value，如果当前线程再迟迟不结束的话，这些 key 为 null 的 Entry 的 value 就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。

    其实，ThreadLocalMap 的设计中已经考虑到这种情况，也加上了一些防护措施：在 ThreadLocal 的get()，set()，remove() 的时候都会清除线程 ThreadLocalMap 里所有 key 为 null 的 value

    处理方法：每次使用完ThreadLocal，都调用它的remove()方法，清除数据

    [深入分析 ThreadLocal 内存泄漏问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/56214714)

38. if/else过多如何优化 

    提前return、策略模式、多态、枚举、使用 Optional、数组小技巧

    

39. 常用设计模式介绍

    工厂模式、单例模式、观察者模式、策略模式、适配器模式、命令模式、装饰者模式、外观模式

    [Java中常用的10种设计模式详解_琪琪的博客-CSDN博客](https://blog.csdn.net/wmq880204/article/details/75106848)

    

40. redis 哨兵与集群的区别 

    哨兵是解决集群的一种方式

    [redis-集群 redis cluster：和哨兵机制的本质区别？ (juejin.cn)](https://juejin.cn/post/6844903872519995400)

    

41. HashMap高并发下会出现什么问题 

    [你知道HashMap在高并发下可能会出现哪些问题吗 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1525791)

    

42. ConcurentHash为什么线程安全 jdk1.8与1.7之前有何区别 

    [ConCurrentHashMap 1.7 和 1.8 的区别 - 简书 (jianshu.com)](https://www.jianshu.com/p/933289f27270)

    

43. 什么场景会出现 栈溢出、内存溢出 CPU消耗率过高 出现的原因是什么，如何排查上述场景 

    类中出现无限引用情况，导致栈的深度超过虚拟机允许深度，占用内存高，无限GC，导致CPU消耗率过高

    临时解决：重启项目，调高-Xms，-Xmx，提高服务器配置

    定位问题：top命令查看最耗CPU的进程，查看该进程中最耗CPU的线程，将线程号转为16进制,同时查看这些线程当前正在干什么，可以看到**最耗CPU的线程都是在进行GC**，用Jmap命令查看当前堆的使用情况，查看gc频率的命令，dump文件分析问题产生原因，定位到出现问题的类位置，查看代码定位问题

    处理问题：优化代码，符合GC回收标准，防止无限引用

    - 内存溢出和内存泄漏的区别
      - 内存溢出 （Out Of Memory）：是指程序在申请内存时，没有足够的内存空间供其使用，出现Out Of Memory。
      - 内存泄露 （Memory Leak）：是指程序在申请内存后，由于某种原因无法释放已申请的内存空间，导致这块内存无法再次被利用，造成系统内存的浪费。memory leak会最终会导致out of memory。
    - 内存溢出分类
      - 栈内存溢出（StackOverflowError）：方法运行的时候栈的深度超过了虚拟机容许的最大深度所致
      - 堆内存溢出(OutOfMemoryError : java heap space)：内存泄露(找出内存泄漏的对象是怎么被GC ROOT引用起来)，内存溢出(程序本生需要的内存大于了我们给虚拟机配置的内存)
      - 持久带内存溢出(OutOfMemoryError: PermGen space)
      - 无法创建本地线程

    [JVM 内存溢出详解（栈溢出，堆溢出，持久代溢出、无法创建本地线程） - myseries - 博客园 (cnblogs.com)](https://www.cnblogs.com/myseries/p/12079757.html)

    [内存溢出+CPU占用过高:问题排查+解决方案+复盘（超详细分析教程）_通往精英的成长之路-CSDN博客](https://blog.csdn.net/zhanghan18333611647/article/details/109255980)

    

44. 解释双亲委派机制 

    https://github.com/chenhj123/chenhj/blob/main/JVM.md

    

45. jdbc为何要打破双亲委派 

    JDBC的Driver接口定义在JDK中，其实现由各个数据库的服务商来提供，比如MySQL驱动包。DriverManager 类中要加载各个实现了Driver接口的类，然后进行管理，但是DriverManager位于 JAVA_HOME中jre/lib/rt.jar 包，由BootStrap类加载器加载，而其Driver接口的实现类是位于服务商提供的 Jar 包，根据类加载机制，当被装载的类引用了另外一个类的时候，虚拟机就会使用装载第一个类的类装载器装载被引用的类。也就是说BootStrap类加载器还要去加载jar包中的Driver接口的实现类。我们知道，BootStrap类加载器默认只负责加载 JAVA_HOME中jre/lib/rt.jar 里所有的class，所以需要由子类加载器去加载Driver实现，这就破坏了双亲委派模型。
    相关文章：

    [阿里面试题：JDBC、Tomcat为什么要破坏双亲委派模型-Java知音 (javazhiyin.com)](https://www.javazhiyin.com/44347.html)

    [聊聊JDBC是如何破坏双亲委派机制的_P19777的博客-CSDN博客](https://blog.csdn.net/P19777/article/details/100829154)

