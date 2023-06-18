## 架构演进
- 单体架构
- 垂直架构
- 分布式架构
- SOA架构
- 微服务架构

## 什么是Dubbo

## Dubbo特点
Dubbo官网


## 源码阅读

### 层级
[Dubbo学习(六) dubbo 架构图 以及调用过程](https://www.cnblogs.com/aspirant/p/9002663.html)
1. Config层：一些基本配置
2. Proxy层：根据配置生成代理，屏蔽上层
3. Registry/Discovery层：服务注册，服务发现层
4. Cluster层：屏蔽多个服务实例的细节，在这一层一般做调用容错以及负载均衡等
5. Monitor层：监控层，监控调用情况等
6. Protocol层：协议层，一般包括Dubbo、
7. Exchange层
8. Transport层：传输层，默认使用netty，服务端使用了NettyServer暴露端口提供服务，客户端使用了NettyClient连接服务
9. Serialize层：序列化层：提供了多种序列化方式，将需要传输的数据（服务方法信息、参数信息和版本等）序列化成传输格式，以及将传输格式转成对应的信息

### 核心类
- DubboBootstrapApplicationListener
- DubboDeployApplicationListener
- ServiceAnnotationPostProcessor
- ReferenceAnnotationBeanPostProcessor

Dubbo3.3分支中

### 启动核心逻辑
@EnableDubbo => @EnableDubboConfig => DubboConfigConfigurationRegistrar # 
DubboDeployApplicationListener#onApplicationEvent ==> onContextRefreshedEvent
DefaultModuleDeployer#start ==> init ==> startSync ==> exportServices ==> referServices

exportServices => export => doExportUrls => doExportUrlsFor1Protocol
referServices => referenceCache.get(rc) => ReferenceConfig#get => ReferenceConfig#init
