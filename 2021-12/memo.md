# 代理
- 静态代理
- 动态代理
    - Java动态代理
    - CGLIB动态代理
    
反射

## Spring

加载bean

### Spring源码深度解析

#### XmlBeanFactory
继承了 `DefaultListableBeanFactory`，内部通过组合 `XmlBeanDefinitionReader` 实现对XML配置资源的读取，
并将XML配置的Document转为BeanDefinition

主要实现：
1. 通过`XmlBeanDefinitionReader`实例读取XML配置文件
    - classpath文件读取为Resource对象
2. 处理和解析配置
    - 对Resource对象再封装一层EncodedResource
    - EncodedResource -> InputStream -> InputSource
    - 获取XML的验证模式(DTD/XSD), 通过`DefaultDocumentLoader` InputSource -> Document
3. 将指定标签的Element解析为`BeanDefinitionHolder`
    - create `BeanDefinitionDocumentReader` 
    - `documentReader` 递归解析XML中`import`, `bean`, `alias`, `beans`等标签, 根据解析结果创建`bdHolder`对象
4. 将`bdHolder`注册到`registry`
    - 将对象注册到`registry`中
    
    
## 加载bean  
AbstractBeanFactory
getBean() -> doGetBean()

1. transformedBeanName(name)

2. DefaultSingletonBeanRegistry getSingleton(beanName)
   从缓存中取（两级缓存(singletonObjects和earlySingletonObjects)）

2.1 `sharedInstance != null && args == null` -> `AbstractBeanFactory getObjectForBeanInstance(sharedInstance, name, beanName, null)`

2.2.1 检测prototype循环依赖
2.2.2 parentBeanFactory
2.2.3 加载依赖的bean
2.2.4 根据scope类型（singleton、prototype、其他）选择createBean
2.2.5 createBean -> doCreateBean
2.2.6 

DefaultSingletonBeanRegistry
    
@startuml
AbstractBeanFactory -> AbstractBeanFactory: getBean()
AbstractBeanFactory -> AbstractBeanFactory: doGetBean()
note left
1. DefaultSingletonBeanRegistry getSingleton(beanName)
从缓存中取（两级缓存(singletonObjects和earlySingletonObjects)）
2. 



end note

@enduml

