
## Faults and Partial Failures
网络的非决定性
### Cloud Computing and Supercomputing
- 超级计算机，很多cpu， 更像一个单系统，局部错就全停机，特别的硬件
- 云cpu，通过网络连接，动态分配， 局部failure, fault-tolerence

对分布式系统，不要期望小概率事件不会发生，怀疑，悲观，偏执狂pay offs。

## Unreliable Networks
- request lost
- request waiting
- 目的节点暂时挂掉了,马上又恢复
- 节点挂掉
- response lost
- response waiting

一般的解决办法是timeout，没有收到response就give up(但是你依然不知道resquest有没有被处理)

### Network Faults in Practice
总之会有各种问题，系统对网络异常是怎么响应的，测试。

### Detecting Faults
- load balancer 需要停止向挂机的节点发送请求
- 在sing-leader模式中，leader挂掉，follower需要被选举为新的leader。

只有收到positive response， 才能保证被正常消费

### Timeouts and Unbounded Delays
超时定多长合适呢

### Network congestion and queueing
- 网络的延迟， queue
- 服务的高负荷， queue
- 如果时虚拟机，另外一台虚拟机使用cpu时，会暂停一段时间
-  TCP flow control

### Synchronous Versus Asynchronous Networks
电话（ISDN网络），每次通话，会建立一个circuit, 分配一个固定的带宽，在circuit的每一个地方都不使用queue, 延迟是固定的。

TCP不是的，是进可能的使用多的带宽。 电话使用的带宽是比较固定的，但是一般网络不是，视频资源，email, 页面等等都不一样，需要尽快传输。

## Unreliable Clocks
NTP协议，可以根据别的server的报道校准时间

### Monotonic Versus Time-of-Day Clocks

### Synchronized clocks for global snapshots
分布式系统产生递增的transactionId
google TrueTime API，会产生一个时间准确的范围，
基于这个范围可以产生一个transactionId, 当两个时间没有overlap时， 没问题， 当有重合时， 就让一个等待一段时间直到提交了一个transaction

### Process Pauses
check时间， 和用时间， 可能间隔很久

JVM GC, 虚拟机切换, 线程切换, 笔记本盖上盖， unix的Ctrl+Z等等

## Knowledge, Truth, and Lies

### The Truth Is Defined by the Majority
单个的节点不可靠

#### The leader and the lock
一个系统需要一些只存在一个的东西

### Byzantine Faults
节点不只会延迟和挂掉，甚至会报告错误的消息。 太空环境中辐射让系统产生任意值。 或者比特币系统， 别的node有可能会欺骗别的node。

要解决这个一般需要硬件支持， 很昂贵

## System Model and Reality
partially synchronous model with crash-recovery faults is generally the most useful model。

### Safety and liveness
考虑一个分布式算法是否正确的很有用的两个model