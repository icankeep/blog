## 分布式ID生成算法
- UUID
- 数据库自增主键
- redis自增key
- snowflake算法（时间戳 + 机器号 + 顺序随机号）
- left算法（双buffer缓存号段，异步获取新号段）

[Leaf——美团点评分布式ID生成系统](https://tech.meituan.com/2017/04/21/mt-leaf.html)