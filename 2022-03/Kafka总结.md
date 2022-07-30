# Kafka总结

## 几种消息队列比较
![](imgs/几种消息队列对比.png)

## Kafka特点
- 吞吐高，单机吞吐量十万级
- 可用性高多副本，分布式主从架构
- 生产者推消费者拉模式
- 消息可靠性，可以做到不丢失
- 消息延迟，ms级
- 可以持久化到磁盘文件，理论上可以无限堆积

## kafka架构图
![](imgs/kafka架构图.png)

****
**Broker**: 一个Kafka集群中的一台服务器就是一个Broker，Broker可以水平无限扩展，同一个topic中的消息可以分布在多个Broker中

**Producer**: 消息生产者，将客户端生产的Message发送到指定topic的leader Partition所在的Broker中。Producer可以通过配置保证写入的消息不会丢失，
Producer同时支持消息异步发送、批量发送

**Consumer**: 消息消费者，Consumer通过向Broker发出一个fetch请求来获取它想要消费的消息，Consumer的每个请求都在消息文件中指定了对应的offset，并接收从该
位置开始的一大块数据，因此，Consumer对于该位置的控制就显得极为重要，并且可以在需要的时候通过回退到某个位置再次消费对应的数据。

**Consumer Group**: 消费者组，同一个消费组中只有一个消费者能收到同一消息

**Topic & Partition**: Topic在逻辑上可以被认为是一个Queue，Kafka中每条消息都必须指定一个Topic，一个Topic中的消息可以分布在集群中的多个Broker中，Consumer
根据订阅的Topic到对应Broker上去拉取消息。为了提升整个集群的吞吐量，物理上一个Topic可以分成多个Partition，每个Partition在磁盘上对对应一个文件夹，该文件夹下存放了这个Partition
的所有消息文件和索引文件。假设有topic1和topic2两个topic，且分别有13个和19个分区，则整个集群会生成32个文件夹。

**ISR**: （In-Sync Replicas）指的是一个Partition中与leader保持同步的replica列表（实际存储的是副本所在Broker的brokerId），这里的保持同步不是指与leader数据保持完全一致，只需在replica.lag.time.max.ms
时间内与leader保持有效连接。

Follower周期性地向leader发送FetchRequest请求，发送时间间隔配置在replica.fetch.wait.max.ms

各Partition的Leader负责维护ISR列表，并将ISR的变更同步至Zookeeper，被移除出ISR的Follower会继续向leader发FetchRequest请求，试图再次跟上Leader重新进入ISR。

通常只有ISR里的成员才可能被选为Leader。当kafka中unclean.leader.election.enable配置为true（默认为false），且ISR中所有副本宕机的情况下，才允许ISR外的副本被选为Leader，此时会丢失部分已应答的数据。

**LEO & HW**:
LEO(log end offset): 即日志末端偏移，指向了副本日志中下一条消息的位移值（即系一条消息的写入位置）
HW(high watermark): 即已同步消息标识，因其类似于木桶效应中短板决定水位高度，顾取名高水位线

**** 

所有高水位线以下都是已备份过得，消费者仅可消费各分区leader高水位线以下的消息，对于任何一个副本对象而言其HW值不会大于LEO值

## 常见的问题

### 消息选择partition策略，以及如何实现消息被顺序消费？
有三种策略，随机策略、轮训策略、按消息键策略和自定义策略

kafka保证同partition中消息有序，如果需要保证消息被有序消费，需要确保需要顺序消费的消息的key相同

### kafka如何保证消息可靠性/不丢失？
https://cloud.tencent.com/developer/article/1722289

kafka 提供三种语义的传递：
- 至少一次 (at least once) 消息不会丢失 ack=all ，但是可能重复投递
- 至多一次 (at most once) 消息可能丢失，但是不会重复投递
- 精确一次 (Exactly Once) 消息不会丢失，也不会重复

生产者，Broker，消费者都是有可能丢数据的。

#### 生产端
##### 生产端丢失数据
即发送的数据根本没有保存到Broker端。出现这个情况的原因可能是，网络抖动，导致消息压根就没有发送到 Broker 端；也可能是消息本身不合格导致 Broker 拒绝接收（比如消息太大了，超过了 Broker 的承受能力）等等。

##### 生产端保证消息不丢失
简单的send发送后不会去管它的结果是否成功，而callback能准确地告诉你消息是否真的提交成功了。一定要使用带有回调通知的 send 方法。

broker一般不会有一个，我们就是要通过多Broker达到高可用的效果。设置 acks = all，表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”，这样可以达到高可用的效果。
- acks=1（默认）：当且仅当leader收到消息返回commit确认信号后认为发送成功。如果 leader 宕机，则会丢失数据。producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果 follower 同步成功之前 leader 故障，那么就会丢失数据。
- acks=0：producer发出消息即完成发送，无需等待来自 broker 的确认。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
- acks=-1（ALL）：发送端需要等待 ISR 列表中所有列表都确认接收数据后才算一次发送完成，可靠性最高，延迟也较大。如果 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，producer 重新发送消息给新 leader 那么会造成数据重复。

##### 注意
Acks=all 就可以代表数据一定不会丢失了吗?当然不是，如果你的 Partition 只有一个副本，也就是一个 Leader，任何 Follower 都没有，因为 ISR 里就一个 Leader，它接收完消息后宕机，也会导致数据丢失。
所以说，这个 Acks=all，必须跟 ISR 列表里至少有 2 个以上的副本配合使用，起码是有一个 Leader 和一个 Follower 才可以

#### Broker端
##### Broker端丢失数据
数据已经保存在broker端，但是数据却丢失了。出现这个的原因可能是，Broker机器down了，当然broker是高可用的，假如你的消息保存在 N 个 Kafka Broker 上，那么至少有 1 个存活就不会丢。

##### Broker端保证消息不丢失

kafka是有限度的保证消息不丢失，这里的限度，是至少一台存储了你消息的的broker。

关注一个leader选举的问题

kafka中有领导者副本（Leader Replica）和追随者副本（Follower Replica），而follower replica存在的唯一目的就是防止消息丢失，并不参与具体的业务逻辑的交互。只有leader 才参与服务，follower的作用就是充当leader的候补，平时的操作也只有信息同步。ISR也就是这组与leader保持同步的replica集合，我们要保证不丢消息，首先要保证ISR的存活（至少有一个备份存活），那存活的概念是什么呢，不仅需要机器正常，还需要跟上leader的消息进度，当达到一定程度的时候就会认为“非存活”状态。

#### 消费端
##### 消费端丢数据
Consumer 程序有个“位移”的概念，表示的是这个 Consumer 当前消费到的 Topic 分区的位置。Kafka默认是自动提交位移的，这样可能会有个问题，假如你在pull(拉取)30条数据，处理到第20条时自动提交了offset，但是在处理21条的时候出现了异常，当你再次pull数据时，由于之前是自动提交的offset，所以是从30条之后开始拉取数据，这也就意味着21-30条的数据发生了丢失。

##### 消费端保证消息不丢失
消费端保证不丢数据，最重要就是保证offset的准确性。我们能做的，就是确保消息消费完成再提交。Consumer 端有个参数 ，设置enable.auto.commit= false，并且采用手动提交位移的方式。如果在处理数据时发生了异常，那就把当前处理失败的offset进行提交(放在finally代码块中)注意一定要确保offset的正确性，当下次再次消费的时候就可以从提交的offset处进行再次消费。consumer在处理数据的时候失败了，其实可以把这条数据给缓存起来，可以是redis、DB、file等，也可以把这条消息存入专门用于存储失败消息的topic中，让其它的consumer专门处理失败的消息。

### 什么时候会repartition？
添加/删除消费者，topic新增partition

### kafka为什么性能这么高？
1. 批量发送消息
1. 消息持久化使用顺序IO，减少磁盘寻道时间，通过索引快速定位消息
1. 磁盘文件交互使用零拷贝；多partition机制，不同partition leader在不同broker中，单机磁盘IO不会成为瓶颈

## kafka如何保证消息Exactly One？
https://www.lixueduan.com/post/kafka/10-exactly-once-impl/

总结：kafka默认幂等性，只支持单分区内&单会话（重启后就无效了）幂等。使用至少发送一次，业务层通过分布式存储存消息id，进行消息过滤

## Kafka是push模式还是pull模式
https://blog.51cto.com/u_15127589/2679155
https://zhuanlan.zhihu.com/p/282993811
https://juejin.cn/post/6844903889003610119

## zmq
https://www.cnblogs.com/yunxintryyoubest/p/14077258.html
https://www.cnblogs.com/mysky007/p/12288729.html
