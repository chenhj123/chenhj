# Spring

### Spring Bean原理

1、Spring Bean生命周期有4个大阶段

- 实例化 Instantiation
- 属性赋值 Populate
- 初始化 Initialization
- 销毁 Destruction

如图：

![SpingBean](https://github.com/chenhj123/chenhj/blob/main/images/SpringBean.png)

直接上代码

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // Instantiate the bean.
   BeanWrapper instanceWrapper = null;
   if (instanceWrapper == null) {
       // 1、实例化阶段！
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }

   // Initialize the bean instance.
   Object exposedObject = bean;
   try {
       // 2、属性赋值阶段！
      populateBean(beanName, mbd, instanceWrapper);
       // 3、初始化阶段！
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }
}
```

销毁是在容器关闭时调用的，详见`ConfigurableApplicationContext#close()`

**首先看两个非常重要的接口**

- BeanPostProcessor
- InstantiationAwareBeanPostProcessor

如图：

![SpringBean](https://upload-images.jianshu.io/upload_images/4558491-dc3eebbd1d6c65f4.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

InstantiationAwareBeanPostProcessor实际上继承了BeanPostProcessor接口

**InstantiationAwareBeanPostProcessor源码分析**

- postProcessBeforeInstantiation调用点，忽略无关代码

  ```java
  @Override
  protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
  throws BeanCreationException {
  
      try {
          // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
          // postProcessBeforeInstantiation方法调用点，这里就不跟进了，
          // 有兴趣的同学可以自己看下，就是for循环调用所有的InstantiationAwareBeanPostProcessor
          Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
          if (bean != null) {
              return bean;
          }
      }
  
      try {   
          // 上文提到的doCreateBean方法，可以看到
          // postProcessBeforeInstantiation方法在创建Bean之前调用
          Object beanInstance = doCreateBean(beanName, mbdToUse, args);
          if (logger.isTraceEnabled()) {
              logger.trace("Finished creating instance of bean '" + beanName + "'");
          }
          return beanInstance;
      }
  
  }
  ```

  可以看到，postProcessBeforeInstantiation在doCreateBean之前调用，也就是在bean实例化之前调用的，英文源码注释解释道该方法的返回值会替换原本的Bean作为代理，这也是Aop等功能实现的关键点。

- postProcessAfterInstantiation调用点，忽略无关代码

  ```java
  protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
  
     // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
     // state of the bean before properties are set. This can be used, for example,
     // to support styles of field injection.
     boolean continueWithPropertyPopulation = true;
      // InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation()
      // 方法作为属性赋值的前置检查条件，在属性赋值之前执行，能够影响是否进行属性赋值！
     if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
           if (bp instanceof InstantiationAwareBeanPostProcessor) {
              InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
              if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                 continueWithPropertyPopulation = false;
                 break;
              }
           }
        }
     }
  
     // 忽略后续的属性赋值操作代码
  }
  ```

  可以看到该方法在属性赋值方法内，但是在真正执行赋值操作之前。其返回值为boolean，返回false时可以阻断属性赋值阶段（`continueWithPropertyPopulation = false;`）。

  关于BeanPostProcessor执行阶段的源码穿插在下文Aware接口的调用时机分析中，因为部分Aware功能的就是通过他实现的!只需要先记住BeanPostProcessor在初始化前后调用就可以了。

**再来看两个重要分类**

- Aware类型的接口
  - BeanNameAware
  - BeanClassLoaderAware
  - BeanFactoryAware
  - EnvironmentAware
  - EmbeddedValueResolverAware
  - ApplicationContextAware
- 生命周期的接口

**Aware调用时机源码分析**

```java
// 见名知意，初始化阶段调用的方法
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {

    // 这里调用的是Group1中的三个Bean开头的Aware
    invokeAwareMethods(beanName, bean);

    Object wrappedBean = bean;

    // 这里调用的是Group2中的几个Aware，
    // 而实质上这里就是前面所说的BeanPostProcessor的调用点！
    // 也就是说与Group1中的Aware不同，这里是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    // 下文即将介绍的InitializingBean调用点
    invokeInitMethods(beanName, wrappedBean, mbd);
    // BeanPostProcessor的另一个调用点
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

    return wrappedBean;
}
```

可以看到并不是所有的Aware接口都使用同样的方式调用。Bean××Aware都是在代码中直接调用的，而ApplicationContext相关的Aware都是通过BeanPostProcessor#postProcessBeforeInitialization()实现的。感兴趣的可以自己看一下ApplicationContextAwareProcessor这个类的源码，就是判断当前创建的Bean是否实现了相关的Aware方法，如果实现了会调用回调方法将资源传递给Bean。
 至于Spring为什么这么实现，应该没什么特殊的考量。也许和Spring的版本升级有关。基于对修改关闭，对扩展开放的原则，Spring对一些新的Aware采用了扩展的方式添加。

BeanPostProcessor的调用时机也能在这里体现，包围住invokeInitMethods方法，也就说明了在初始化阶段的前后执行。

关于Aware接口的执行顺序，其实只需要记住第一组在第二组执行之前就行了。每组中各个Aware方法的调用顺序其实没有必要记，有需要的时候点进源码一看便知。

**简单的两个生命周期接口**

至于剩下的两个生命周期接口就很简单了，实例化和属性赋值都是Spring帮助我们做的，能够自己实现的有初始化和销毁两个生命周期阶段。

1. InitializingBean 对应生命周期的初始化阶段，在上面源码的`invokeInitMethods(beanName, wrappedBean, mbd);`方法中调用。
    有一点需要注意，因为Aware方法都是执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展Spring的常用方式。
    除了实现InitializingBean接口之外还能通过注解或者xml配置的方式指定初始化方法，至于这几种定义方式的调用顺序其实没有必要记。因为这几个方法对应的都是同一个生命周期，只是实现方式不同，我们一般只采用其中一种方式。
2. DisposableBean 类似于InitializingBean，对应生命周期的销毁阶段，以ConfigurableApplicationContext#close()方法作为入口，实现是通过循环取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 。感兴趣的可以自行跟一下源码。

------

参考资料

1. ### Spring AOP


[chenhj/Sping AOP.md at main · chenhj123/chenhj (github.com)](https://github.com/chenhj123/chenhj/blob/main/Sping AOP.md)

2. ### Spring Annotation

[chenhj/Spring Annotation.md at main · chenhj123/chenhj (github.com)](https://github.com/chenhj123/chenhj/blob/main/Spring Annotation.md)

3. ### Spring Bean

   [请别再问Spring Bean的生命周期了！ - 简书 (jianshu.com)](https://www.jianshu.com/p/1dec08d290c1)

4. ### Spring 系列之beanFactory 与 ApplicationContext

   [Spring系列之beanFactory与ApplicationContext - 平凡希 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiaoxi/p/5846416.html)