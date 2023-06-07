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
