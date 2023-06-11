> 参考博客：https://xiaolincoding.com/redis/base/redis_interview.html#%E8%AE%A4%E8%AF%86-redis

## 高性能原因
- C 语言实现，虽然 C 对 Redis 的性能有助力，但语言并不是最核心因素。
- 纯内存 I/O，相较于其他基于磁盘的 DB，Redis 的纯内存操作有着天然的性能优势。
- I/O 多路复用，基于 epoll/select/kqueue 等 I/O 多路复用技术，实现高吞吐的网络 I/O。
- 单线程模型，单线程无法利用多核，但是从另一个层面来说则避免了多线程频繁上下文切换，以及同步机制如锁带来的开销。

![why-redis-so-fast](imgs/why-redis-so-fast.png)

## 基本数据类型
- string => SDS
- hash => 压缩列表或哈希表
- set => 整数集合或哈希表
- zset => 压缩列表或跳表
- list => quicklist  |  压缩列表或双向链表

## 线程模型
对于部分耗时操作，会由后台线程处理，后台线程相当于消费者，从队列中取对应任务

主线程通过IO多路复用epoll，以及任务分发，做到单线程内高性能切换

## Redis持久化机制
- AOF => AOF重写、AOF刷盘
- RDB
- 混合持久化

AOF重写过程：fork子进程，子进程中有父进程数据副本，如果父进程有写操作，会触发写时复制，若此时有新增写命令，会将AOF命令添加至AOF缓冲区，等待重写过程
结束后，再补充到AOF中

RDB快照：分为save和bgsave  fork子进程操作

### 为什么是fork子进程而不是线程？
fork进程，利用操作系统能力就可以共享父进程内存，通过父进程有写操作时也会触发写时复制，通过线程还需要加锁控制并发问题，影响性能。

## Redis淘汰机制

## 缓存雪崩
大量缓存同时失效

每个key设置过期时间时增加一个随机数

## 缓存击穿
我们的业务通常会有几个数据会被频繁地访问，比如秒杀活动，这类被频地访问的数据被称为热点数据。

如果缓存中的某个热点数据过期了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是缓存击穿的问题。

- 互斥锁方案（Redis 中使用 setNX 方法设置一个状态位，表示这是一种锁定状态），保证同一时间只有一个业务线程请求缓存，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。
- 不给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据准备要过期前，提前通知后台线程更新缓存以及重新设置过期时间；

## 缓存穿透
大量不存在的key直接打到数据库，导致数据库挂了

通过布隆过滤器先进行一层过滤，判断数据是否存在

## 缓存预热
...

## 缓存更新策略
- Cache Aside 旁路缓存：读时读缓存，不存在读数据库后更新缓存，更新时先更新数据库，更新成功后删除缓存
- 读穿/写穿 Read Through/ Write Through
- Write Back 写回策略 （CPU缓存和文件系统的Page Cache等）

## 缓存降级
... 设置降级策略，直接返回默认值

## 集群、哨兵
- Cluster  哈希槽 一致性哈希
  - [Redis集群使用一致性哈希和哈希槽区别](https://www.jianshu.com/p/3f1c801b22ff)
- Sentinel 哨兵，主从切换

## Redis后台线程
我们虽然经常说 Redis 是单线程模型（主要逻辑是单线程完成的），但实际还有一些后台线程用于执行一些比较耗时的操作：

- 通过 bio_close_file 后台线程来释放 AOF / RDB 等过程中产生的临时文件资源。
- 通过 bio_aof_fsync 后台线程调用 fsync 函数将系统内核缓冲区还未同步到到磁盘的数据强制刷到磁盘（ AOF 文件）。
- 通过 bio_lazy_free后台线程释放大对象（已删除）占用的内存空间.

[Redis后台线程有哪些](https://juejin.cn/post/7102780434739626014)

## Redis多线程网络模型
[Redis 多线程网络模型全面揭秘](https://segmentfault.com/a/1190000039223696)

## 常见问题

### Redis中的hash结构和Java HashMap扩容的差异？
Redis中hash扩容是渐进式扩容，可以把扩容耗时均摊给每次请求，扩容期间同时存在两个hashtable，对redis的查询、删除、修改、插入同时会将rehashidx处的key迁移到新的table，
当rehashidx达到原先table最大长度时，扩容完成...

HashMap是直接完成，扩容后将所有元素复制到新的hashtable

### Redis如何实现一个无限大的数据库
Redis集群，通过slot槽位，分配不同的槽给不同的redis server，key通过CRC算法计算存储的对应槽位

### aof文件优化策略？
aof文件重写，通过已有key生成命令

### 跳表数据结构，插入数据时怎么选择哪一层？
https://www.jianshu.com/p/9d8296562806
https://blog.csdn.net/m0_49834705/article/details/112465244
https://www.jianshu.com/p/54d37710b2a6

### 为什么Redis选择使用跳表而不是红黑树来实现有序集合？
Redis 中的有序集合(zset) 支持的操作：
- 插入一个元素
- 删除一个元素
- 查找一个元素
- 有序输出所有元素
- 按照范围区间查找元素（比如查找值在 [100, 356] 之间的数据）

其中，前四个操作红黑树也可以完成，且时间复杂度跟跳表是一样的。但是，按照区间来查找数据这个操作，红黑树的效率没有跳表高。按照区间查找数据时，跳表可以做到 O(logn) 的时间复杂度定位区间的起点，然后在原始链表中顺序往后遍历就可以了，非常高效。

### Redis 是如何判断数据是否过期的呢？
Redis 通过一个叫做过期字典（可以看作是 hash 表）来保存数据过期的时间。过期字典的键指向 Redis 数据库中的某个 key(键)，过期字典的值是一个 long long 类型的整数，这个整数保存了 key 所指向的数据库键的过期时间（毫秒精度的 UNIX 时间戳）。

![](imgs/redis-expire.png)

### 如何实现分布式延时队列
https://juejin.cn/post/6844904016434970637

### Redis常见阻塞场景
[参考资料](https://github.com/Snailclimb/JavaGuide/blob/main/docs/database/redis/redis-common-blocking-problems-summary.md)

概要：
- 执行复杂度较高的命令（O(n)）
- 使用save创建RDB快照时
- AOF写日志时
- AOF刷盘时（fsync）
- AOF重写时
- 大key问题（查询/插入/删除大key）
- 集群扩容
- swap交换，控制redis内存和机器内存，降低swap交换优先级

### 使用Redis实现一个分布式限流器
思路：
通过redis lua脚本往redis中set两个值，一个为有过期时间的限流token个数key，value为容量，另一个为有过期时间的限流时间戳key，value为上次操作时间戳，

根据配置的token生成速率以及总容量值，每次请求时都会根据速率在剩余的token中添加上新生成的个数，如果token个数满足，则返回通过，否则不通过

参考的博客：[Redis 分布式限流器](https://mp.weixin.qq.com/s/kyFAWH3mVNJvurQDt4vchA)

spring-cloud-gateway限流最佳实践
- [Redis限流器lua命令](https://github.com/spring-cloud/spring-cloud-gateway/blob/4aeccc736e98228105fc623bc9f1bf78014e18ce/spring-cloud-gateway-server/src/main/resources/META-INF/scripts/request_rate_limiter.lua#L1)
- [执行Redis lua命令](https://github.com/spring-cloud/spring-cloud-gateway/blob/4aeccc736e98228105fc623bc9f1bf78014e18ce/spring-cloud-gateway-server/src/main/java/org/springframework/cloud/gateway/filter/ratelimit/RedisRateLimiter.java#L253)

扩展一下，如果单个限流key可能有百万QPS，Redis顶不住压力，怎么处理？

利用分治的思想，server端每次批量请求一批token，返回的token信息中存在过期时间戳，当过期时才从redis取，否则每次都在server端做单机限流

### Redis实现分布式锁
核心思路还是往redis中set一个key，这个key就是这个分布式锁的唯一标识，而value需要是可以判断锁持有方是否是当前线程的唯一ID（一般可以设置为UUID），

需要额外考虑的有几点：
- 需要有NX标识，当不存在时才set，确保当前线程唯一获取到锁的线程
- 需要有EX标识，当持有锁的线程因为各种意外原因未释放锁时，对应的key也能过期失效，不会造成锁一直未被释放
- 有过期就需要有续期动作，在线程持久锁过程中，锁不能因为过期而被释放，需要有后台自动执行续期的任务
- 释放锁时需要lua脚本（确保原子）比较value和传递的值是否一致，否则可能其他的线程释放了不是它本身持有的锁

一般用现成的框架：Redisson

[分布式锁常见问题总结](https://javaguide.cn/distributed-system/distributed-lock.html#%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E4%BB%8B%E7%BB%8D)

### 使用缓存的方式
- 旁路缓存 cache aside
- 读写穿透 write through
- 异步缓存写入 write back

[3种常用的缓存读写策略详解](https://javaguide.cn/database/redis/3-commonly-used-cache-read-and-write-strategies.html#read-write-through-pattern-%E8%AF%BB%E5%86%99%E7%A9%BF%E9%80%8F)

[阿里云栖号分享的一篇缓存文章](https://mp.weixin.qq.com/s/JyrrRjLEvwfPeDEVUfy_VQ)