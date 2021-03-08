# JVM核心技术

### 类的生命周期

1. 加载（Loading）：找Class文件
2. 验证（Verification）：验证格式、依赖
3. 准备（Preparation）：静态字段、方法表
4. 解析（Resolution）：符号解析为引用
5. 初始化（Initialization）：构造器、静态变量赋值、静态代码块
6. 使用（Using）
7. 卸载（Unloading）

### 三类加载器：

1. 启动类加载器（BootstrapClassLoader）
2. 扩展类加载器（ExtClassLoader）
3. 应用类加载器（AppClassLoader）

### 双亲委托机制

![类加载](images\classloader.png)

- 其中红色的箭头代表向上委托的方向，如果当前的类加载器没有从缓存中找到这个class对象，就会请求父加载器进行操作。直到启动类加载器。

- 而黑色的箭头代表的是查找方向，若`Bootstrap ClassLoader`可以从`%JAVA_HOME%/jre/lib`目录或者-Xbootclasspath指定目录查找到，就直接返回该对象，否则就让`ExtClassLoader`去查找。

- `ExtClassLoader`就会从`JAVA_HOME/jre/lib/ext`或者`-Djava.ext.dir`指定位置中查找，找不到时就交给`AppClassLoader`，`AppClassLoader`就从当前工程的bin目录下查找

- 若还是找不到的话，就由我们自定义的`CustomClassLoader`查找，具体查找的结果，就要看我们怎么实现自定义ClassLoader的`findClass`方法了。

### 双亲委托作用

1. 避免重复加载，加载过了直接从缓存里边读取
2. 安全考虑，可自定义一个String类来替代系统的String类，造成安全隐患

### JVM内存结构

![内存结构](images\memory_structure.png)

上图有一个地方出现错误，元数据区Metaspace与Compressed Class Space存放class信息之间有交叉

问题：为什么把-Xms和-Xmx的值设置成一样

-Xms：指定虚拟机堆内存初始化大小；-Xmx：指定虚拟机堆内存最大值大小

把两者设置为一致，是为了避免频繁扩容和GC释放堆内存造成的系统开销/压力