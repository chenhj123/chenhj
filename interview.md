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
   
9. 代码如下

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
   