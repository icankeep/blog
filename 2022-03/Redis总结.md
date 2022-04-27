## Redis持久化机制
AOF和RDB

## Redis淘汰机制

## 缓存雪崩
大量缓存同时失效

每个key设置过期时间时增加一个随机数

## 缓存击穿

## 缓存穿透
大量不存在的key直接打到数据库，导致数据库挂了

通过布隆过滤器先进行一层过滤，判断数据是否存在

## 缓存预热
...

## 缓存更新
先更新数据库，更新成功后删除缓存

## 缓存降级
... 设置降级策略，直接返回默认值

## Redis中的hash结构和Java HashMap扩容的差异？
Redis中hash扩容是渐进式扩容，可以把扩容耗时均摊给每次请求，扩容期间同时存在两个hashtable，对redis的查询、删除、修改、插入同时会将rehashidx处的key迁移到新的table，
当rehashidx达到原先table最大长度时，扩容完成...

HashMap是直接完成，扩容后将所有元素复制到新的hashtable

## Redis如何实现一个无限大的数据库
Redis集群，通过slot槽位，分配不同的槽给不同的redis server，key通过CRC算法计算存储的对应槽位

## rdb为什么fork进程而不是线程？
fork子进程可以读取到原先进程内存中的数据，线程的话可能会导致锁争用的情况

## aof文件优化策略？
aof文件重写，通过已有key生成命令

## 跳表数据结构，插入数据时怎么选择哪一层？
https://www.jianshu.com/p/9d8296562806
https://blog.csdn.net/m0_49834705/article/details/112465244
https://www.jianshu.com/p/54d37710b2a6

## 如何实现分布式延时队列
https://juejin.cn/post/6844904016434970637
