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

