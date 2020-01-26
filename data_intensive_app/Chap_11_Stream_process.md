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

