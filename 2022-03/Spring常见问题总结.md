1. Spring如何解决循环依赖问题
    通过多层缓存提前暴露来解决
2. FactoryBean和BeanFactory区别
    - BeanFactory实际是一个容器，ApplicationContext等都实现了BeanFactory接口，定义了getBean等方法
    - FactoryBean实际是一个Bean，可以在容器通过getBean获取，不过获取的是这个bean调用getObject方法后得到的对象，相当于在bean创建的时候加了更多可配置操作的空间，如果需要获取真实的Bean，
需要在name前面添加&
    https://juejin.cn/post/6844903967600836621
3. BeanFactory和ApplicationContext区别
    https://www.cnblogs.com/xiaoxi/p/5846416.html
4. Spring Bean的生命周期，是如何被管理的?

![Spring Bean的生命周期](imgs/Spring%20Bean的生命周期.png)

5. Spring Bean的加载过程是怎样的？

6. SpringBoot启动过程

6 零拷贝
https://juejin.cn/post/6995519558475841550


- BeanDefinitionRegistryPostProcessor和BeanFactoryPostProcessor差别


在Spring中，常被用于扩展的接口有以下几个：

BeanPostProcessor：Bean后置处理器接口，它定义了在Spring容器实例化bean之后、初始化bean之前和之后执行的回调方法。通过实现BeanPostProcessor接口，我们可以在bean实例化和初始化的过程中对bean进行自定义的处理，例如修改bean的属性、添加代理等。

BeanFactoryPostProcessor：Bean工厂后置处理器接口，它定义了在Spring容器实例化bean之后、初始化bean之前执行的回调方法。通过实现BeanFactoryPostProcessor接口，我们可以在bean实例化之前对bean的定义进行修改，例如添加新的属性、修改属性值等。

ApplicationContextAware：应用上下文感知接口，它定义了一个setApplicationContext方法，用于在bean实例化之后自动注入应用上下文对象。通过实现ApplicationContextAware接口，我们可以在bean中获取应用上下文对象，从而实现一些高级的功能，例如获取其他bean的引用、获取环境变量等。

InitializingBean：初始化接口，它定义了一个afterPropertiesSet方法，用于在bean属性设置完成之后执行一些初始化操作。通过实现InitializingBean接口，我们可以在bean初始化的过程中执行一些自定义的初始化操作，例如检查bean的状态、初始化一些资源等。

DisposableBean：销毁接口，它定义了一个destroy方法，用于在bean销毁之前执行一些清理操作。通过实现DisposableBean接口，我们可以在bean销毁的过程中执行一些自定义的清理操作，例如释放一些资源、关闭一些连接等。
FactoryBean
通过实现这些接口，我们可以在Spring容器中对bean进行更加细粒度的控制和扩展，实现一些高级的功能和自定义的处理逻辑


