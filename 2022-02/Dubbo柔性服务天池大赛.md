## 题目
- [题目链接](https://tianchi.aliyun.com/competition/entrance/531923/information?spm=5176.12281976.0.0.562de0b1WAqL4H)
- [优秀的赛后解析分享](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.3.569861a2i7qTpq&postId=326699)
 
### 部署的服务架构
![image.png](https://img.alicdn.com/imgextra/i2/O1CN01Hp8HO420vkIeZD3QP_!!6000000006912-2-tps-1224-558.png)

部署架构说明：

- 所有程序均在不同的 docker 容器中运行，每个容器都独占运行在不同的虚拟机上
- Gateway 负责将请求转发至 Provider
- Provider 处理请求返回响应
- Gateway 和 Provider 之间采用 Nacos 注册中心进行服务发现
- 选手需要设计实现 Gateway 选择 Provider 的 Cluster、LoadBalance 算法


测试过程：

1. PTS 作为压测请求客户端向 Gateway（Consumer） 发起 HTTP 请求，Gateway（Consumer） 加载用户实现的负载均衡算法选择一个 Provider，Provider 处理请求，返回结果。
1. 每个 Provider 的服务能力 (处理请求的速率) 都会动态变化：
    1. 三个 Provider 的每个 Provider 的处理能力会**随机变动**以模拟超售场景
    1. 三个 Provider 任意一个的处理能力**都小于**总请求量
    1. 三个 Provider 的会有**一定比例**的请求处理超时（5000ms）
    1. 三个 Provider 的每个 Provider 会**随机离线**（本次比赛不依赖 Nacos 的健康检查机制，也即是无地址更新通知）
3. 评测分为预热和正式评测两部分，预热部分不计算成绩，正式评测部分计算成绩。
3. 正式评测阶段，PTS 以固定 RPS 请求数模式向 Gateway 发送请求，1分钟后停止；
3. 以 PTS 统计的成功请求数和最大 TPS 作为排名依据。成功请求数越大，排名越靠前。成功数相同的情况下，按照最大 TPS 排名。


### 消费端请求方式


在 Dubbo 中， `Filter` 被设计用来拦截和过滤单次请求，基于这个实现，用户和开发者可以在不改变核心框架的情况下，非常方便的嵌入自己的逻辑来影响请求行为和请求数据。


从 3.0 版本开始，在保持原有 `Filter` 拦截语义的情况下，框架在消费端引入了新的拦截扩展点 `ClusterFilter`，用于在选址之前拦截请求，选手可以自行选择采用 `ClusterFilter` 或 `Filter` 进行请求拦截。
​

在 `ClusterInvoker` 中将会传入全部 Provider 信息，选手需要基于一定规则选择最佳 Provider 进行调用或者拒绝请求。


![image.png](https://img.alicdn.com/imgextra/i4/O1CN01Dt2tDC1hztBNo0Uyc_!!6000000004349-2-tps-752-175.png)


### 服务端处理方式
![image.png](https://img.alicdn.com/imgextra/i4/O1CN01V3fDZC1RfhsXlGLuF_!!6000000002139-2-tps-1574-226.png)
在服务端收到请求后会经过一系列的过滤链，最终调用到具体业务实现上。选手可以选择通过 Filter 对请求状态进行监控，亦或者是拒绝请求。
​

当服务端需要将容量信息通知消费者时，仅允许使用 `Result appResponse` 中的 attachment 进行传递，不允许对 Apache Dubbo 自有的协议体进行任何修改。

## 大意解析
服务分为压测端 -> client端 -> provider端


我们可以在client端实现自己的负载均衡算法、超时策略和节点选择策略，在provider端对自身服务容量评估进行限流等操作


主要从以下几个点写代码：
1. 计算最佳超时时间：成功请求花费的时间的滑动窗口（P70 P80 P90 P95 P99）`percent / elapsed` 取最优，取成功请求花费平均时间`t * 1.x`
2. 负载均衡策略：按照每个invoker的tps计算权重
3. provider端限流算法，调节参数测试算法：进入filter后，获取限流token，没获取到直接返回，获取到的话直接调用服务（限流值动态计算，预热一分钟阶段梯步递增，先快后慢，锁定最佳值）
4. provider会随机离线：离线心跳探活，超过一定时间provider端没有响应，provider状态定为离线，负载均衡降低该节点权重，大概几百个请求探活一次
