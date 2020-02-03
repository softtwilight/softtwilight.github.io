# chapter_12
- 数据是无限的
- 实时性

## Transmitting Event Streams
batch processing 的输入输出是文件。
流是更小的，一般是record，称为event。相关的event，一般组合进一个topic或stream。

### Message System
publish/subscribe模型。
一般message系统都需要考虑两个问题：
- 如果producer生产的能力高于consumer消费的能力，怎么办
    1. drop message
    2. buffer message （如果buffer满了呢
    3. 限流

- 如果节点挂掉或者网络问题，会丢消息吗

#### producer直接发送消息到consumer
低延迟，一般需要应用代码来意识到message丢失（金融系统），一般需要两者同时在线

#### message brokers（MQ
producers write message to brokers, and consumers receive message from brokers.
对生产者消费者同时ok的要求降低了，把消息持久化的任务也转移到broker身上。

#### multiple consumers
- load balanceing, 消息只发送到其中一个consumer
- Fan-out，消息发送给所有consumer

#### Acknowledgments and redelivery
client必须告诉broker，已经处理完了消息，broker才会把message删去。
redelivery 加上 load balanceing 会无法保证message的顺序。

### Partitioned Logs
message一般是没有持久性的，对于batch processing，假设就是input不会变，所以当处理失败的时候会重试不影响output，所以有了log-based message。
日志是partitioned，每条message被分配一个递增的id，在单节点上是有序唯一的。Apache Kafka , Amazon Kinesis Streams, and Twitter’s DistributedLog 是用的这种架构。

log型的message很好支持Fan-out模式，read log不会影响其他reader。
load-balance可以是一个节点读一个partition。

对于message处理很耗时，需要并行处理，且对顺序要求不高是，JMS类型很适合。
对于message处理很快，吞吐量很大，要求顺序的场合，log-based message很适合。

#### consumer offset
因为consumer只要保存一个消息的id（offset），对比log，就知道哪些是消费的，哪些是没有消费的，所以不需要broker收到client的acknowledgment，只需要定时记录consumer的offset。提高了吞吐量。
offset 很像 log sequence number。
如果consumer挂掉，重启后会从broker里记录的offset开始重新消费消息，但是这些消息是可能被消费过的，而消费后offset还没有反映在broker。

因为consumer是根据offset消费消息的，很容易决定从哪里开始消费，（JMS办不到），所以很适合batch process一样处理某一天的数据，重复消费等。

#### 硬盘使用
不会无限制的写log，一般会清除很久的segment，如果consumer消费落后很多很多，是有可能丢消息的。

## DataBase and Stream
### Keep System in Sync
database， cache， search index, data warehouse. 复杂应用会用到这些存储数据，保证数据的Sync。
batch process。 也可以双写。 双写可能会存在不一致的情况。A1 B1 B2 A2， 可能A B 写 系统 1 2 的操作顺序会是这样， 找出1 2 系统数据不一致。
同时写两个系统也会有atomic commit的问题，是very expensive的。

### Change Data Capture
观察到数据的变化，并将变化以一定的形式发送给别的系统。
压缩日志。(同一条数据的多次操作，只保留最新的，效果类似于snapshot)

### Event Source
DDD(domain-driven design的概念)，将所有的变化封装为一个change event。关注事件本身，而不是事件产生的结果。
用户一般期待的是现在的状态，所以只有记录时不够的。比如大家只会关心购物网站上剩余的口罩数，而不是关心上架和已销售数目。
一般用event sourcing的应用都要保存一些时间点的snap-shot，这样不用读取所有的event 历史。

### State, Streams, and Immutability
当前状态时event stream的积分。（人生是不是也是这样。。。
event sourcing最大的缺点时异步的，写之后可能读到的还是旧数据。

## Processing Message
restart a task and sort 不现实， stream 是endless的

### use of Stream processing

### Reasoning About time
会经常使用time window， 类似最近5分钟的平均流量。"最近5分钟"比看起来更复杂一些。
看event中的timestamp（决定性的 和 系统时间 （简单，需要event发生的时间和处理时间很小

如果用event 时间， 有一个问题是，你不知道在一个windows里面，是不是还有别的event。（如果一段时间没收到，就认为完成，再来的就忽视或者发布一个更正）。
采集多个timestamp， event 发生的时间， client发送的时间， server 接受的时间

type of windows: //for aggregations
- Tumbling windows(翻滚), fixed length, 一个event只属于一个window 1- 2, 2-3, 3-4 ... 
- hopping windows(跳跃)，fixed length, 但是允许overlap，目的是统计更平顺。1-5， 2-6， 3-7...
- sliding windows, 某个时间长度范围内发生的所有事件， 1.5-2.5... 将事件放入一个buffer， 然后移除过期的事件， buffer内的就是过期时间范围内所有的事件了。
- session windows, 没有固定事件，记录用户一次登录的所有事件。

### Stream join

#### stream-stream join
搜索stream 和 点击 steam join，获得搜索质量评估的stream

### Fault Tolerance
exactly-once processing

- microbatch and checkpoint 如果涉及周边系统，回到checkpoint点还是会有问题
