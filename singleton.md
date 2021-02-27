# 单例模式

1. 饿汉单例模式

   ```java
   public class SingleDemo1 {
       //类初始化时，立即加载这个对象,由于加载类时，天然得线程时安全得；
       private static SingleDemo1 instance = new SingleDemo1();
   
       private SingleDemo1(){}
   
       //方法没有同步，调用效率高
       public static SingleDemo1 getInstance(){
           return instance;
       }
   }
   ```
   
2. 懒汉式单例

   ```java
   public class SingleDemo2 {
       //类初始化时，不初始化这个对象(延时加载，真正用的时候再创建)
       private static SingleDemo2 instance;
   
       private SingleDemo2(){}
   
       //方法没有同步，调用效率高
       public static synchronized SingleDemo2 getInstance(){
           if (instance==null){
               instance = new SingleDemo2();
           }
           return instance;
       }
   }
   ```
   
3. 双重检查锁实现单例模式

   ```java
   public class Singleton {  
       private volatile static Singleton singleton;  
       private Singleton (){}  
       public static Singleton getSingleton() {  
       if (singleton == null) {  
           synchronized (Singleton.class) {  
           if (singleton == null) {  
               singleton = new Singleton();  
           }  
           }  
       }  
       return singleton;  
       }  
   }
   ```

4. 静态内部类实现单例模式

   ```java
   //静态内部类实现单例模式
   //线程安全，调用效率高，并且实现了延时加载
   public class SingleDemo4 {
       private static class SingletonClassinstance{
           private static final SingleDemo4 instance = new SingleDemo4();
       }
   
       public static SingleDemo4 getInstance(){
           return SingletonClassinstance.instance;
       }
   
       private SingleDemo4(){}
   }
   ```

5. 枚举单例(没有延时加载)

   ```java
   //枚举单例(没有延时加载)
   public class Singleton {
       private Singleton(){}
       private enum SingletonEnum{
           INSTANCE;
           private final Singleton instance;
           SingletonEnum(){
               instance = new Singleton();
           }
           private Singleton getInstance(){
               return instance;
           }
       }
   
       public static Singleton getInstance(){
           return SingletonEnum.INSTANCE.getInstance();
       }
   }
   ```