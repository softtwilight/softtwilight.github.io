
最终一致性， 当停止写， 并等待一段不定的时间， 数据会一直。非常弱的一致性保证。

一致性关心的是不同replicas之间的状态。

## Linearizability
让系统看起来只有一份数据，并且所有的操作是原子性的。

### Implementing Linearizable Systems
- 如果是单leader的系统，读也经过leader，那么具有潜在的linearizability（比如能否确实正确的leader）
- 一致性算法（ZooKeeper, etcd) 
- 多leader的系统基本不是linearizable.一般是报告冲突，然后解决
- leaderless的系统基本不是linearizability

- 如果需要linearizability, 那么当节点disconnected的时候，要么等待连接恢复，要么返回错误，不能继续处理请求。（不可用
- 如果不需要保证linearizability, 那么可以独立的处理请求，即使disconnected, 保证了可用
- CAP其实比较容易让人误解，P一般不是个选择，C和A是选择，而且可用性也跟平常所用的系统可用性有不一样的含义

### Linearizability and network delays
实际上很少的系统是真正的Linearizability， 操作系统的RAM， 多进程写内存， 因为cache的存在，实际不是linearizability的，是出于性能的考虑，而不是CAP中who。

已经有证明，不能用更有效的方法来实现Linearizability

## Ordering Guarantees

### order and casuality
- 前有问题才有答案
- 更新前必须有数据

causally consistent， A -> B -> C, 如果读到C， 那么一定也可以读到AB。causally order 不等于 total order（对于任意两个事件，你都知道先后顺序）。

对于linearizability系统， 因为相当于只有一份数据，每个操作都是atomic的，所以是total order的。
对于并发的情况，A B同时发生， A不是B的原因，也不是结果， 分叉并合并，不是一条timeline（git 的分支流

不一定要线性系统才能保证casuality， 这是现在的研究方向，没有成熟的应用，关键是要记录casuality，happen-before。 Version-vector

### Sequence number ordering
记录因果的顺序有时候也是不实际的，比如读很多条数据才写一条数据，因很多导致难以记录，可以采用seq num，类似一个counter， 每一个操作赋予一个递增的唯一的数。

#### Lamport timestamps
consistent with causality。 every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request。系统报道最大的counter，如果比节点自身的大，就更新为maximum counter
每个节点有个自己的timpstamp, 再加上节点ID

这是一种事后总结出来的顺序，对于其中的节点，其实是不知道自己的counter是不是最大的，所以对于某些场景，比如唯一username，其实节点是不能决定的。

### Total Order Broadcast
需要两个前提：

- Reliable delivery   消息不会丢失
- Totally ordered delivery   消息以同样的顺序发送给各个节点

Zookeeper 和 etcd 实现了Total Order Broadcast  
“state machine replication”
像是写日志

线性加CAS（或者atomic的incrementAndGet）和Total Order Broadcast 都等价于consensus

## Distributed Transactions and Consensus
一些场景：
- leader election
- atomic commit

2pc(两阶段提交是一种consensus算法，但不那么好)
coordinator（transaction manager)
phase1： coordinator 发送prepare请求到每一个节点， 节点返回yes/no
phase2： 如果都返回yes， coordinator发送commit请求。 如果有no或者超市， 发送abort请求

为什么2pc能保证提交的原子性能，请求不是一样会丢失吗？
- 当应用开始分布式事务时， 向coordinator请求一个唯一的transaction id
- 应用在每一个节点上开始事务，完成所有读写，如果出错或超时，废弃。
- coordinator发送prepare请求到所有参与者，如果有错误或超时，废弃所有事务。
- 当参与者接受到prepare请求时，会检查一切情况，保证可以commit（数据写入硬盘不会丢失，空间足够，检查了conflict和数据限制等），回复Yes意味着只要收到commit请求，就一定会commit成功，已经没有abort的权利了。这一步很关键。
- coordinator通过收到的回复决定是否commit，当决定了之后要 将决定transaction log，这个时间点成为commit point
- coordinator发送commit或abort 请求，如果超时，就一直重复直到成功，没有回头路了。

### Distributed Transactions in Practice
2pc性能差，mysql的例子，分布式事务至少慢10倍
同质的分布式事务一般可以profrom well（mysql 事务 和mysql 事务），异质的就更差了（mysql 和 MQ）
异质事务协议

#### XA transactions （eXtended Architecture）
In Java， 用JTA实现了XA， 又被很多JDBC 和 JMS 支持

#### Holding locks while in doubt
为了保证read-commited，或者更高的隔离级别，一般数据会上锁，当数据库在等待coordinator恢复的过程中，会一直持有锁。其他访问数据的事务会等待。

当coordinator 的transaction log 丢失或者其他因bug产生问题时，时很棘手的，coordinator不知道时commit还是abort，参与者还在持锁等待，重启是没用的，因为2pc意味着承诺重启也会等待。这时候一般的xa实现会有一种应急方案，丢失atomic性，保证完成事务。

#### XA的问题
- coordinator本身可能挂掉，很多系统没有对coordinator分布，（coordinator分布是不是需要额外的coordinator？
- coordinator本身其实是一个database，有状态的，有的应用server是无状态的，可以随意增减节点，引入XA就不是无状态了。
- 不支持SSI
- 当一个节点失败的时候，会一直重试，有点违背单点失败的分布式初衷。


### Fault-Tolerant Consensus
a consensus algorithm 必须满足的性质：
- Uniform agreement 最终所有节点决定一致
- Integrity  决定后不反悔
- Validity 不止有一个节点决定了值是v
- Termination   Every node that does not crash eventually decides some value.保证最终会做出决定（2pc就不能保证这个，比如coordinator被陨石砸中了）

 consensus 算法必须要求至少有大多数的节点是ok的。同时也要没有拜占庭将军问题（有节点报告错误消息）

 ### Consensus algorithms and total order broadcast
常见算法：
- Viewstamped Replication
- Paxos
- Raft
- Zab

total order broadcast is equivalent to repeated rounds of consensus

#### Epoch numbering and quorums
保证在一个时期内（Epoch number）是只有一个leader的

we have two rounds of voting: once to choose a leader, and a second time to
vote on a leader’s proposal. The key insight is that the quorums for those two votes
must overlap: if a vote on a proposal succeeds, at least one of the nodes that voted for
it must have also participated in the most recent leader election . Thus, if the
vote on a proposal does not reveal any higher-numbered epoch, the current leader
can conclude that no leader election with a higher epoch number has happened, and
therefore be sure that it still holds the leadership. It can then safely decide the pro‐
posed value.


### Membership and Coordination Services
ZooKeeper play an important role in providing an “outsourced” consen‐
sus, failure detection, and membership service。

