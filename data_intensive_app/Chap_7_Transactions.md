A transaction is a way for an application to group several reads and writes together into a logical unit. to simplify the programming model for applications accessing a database.

## The Slippery Concept of a Transaction

### ACID
Atomicity, Consistency, Isolation, and Durability. 
- Atomicity 错误时废弃所有之前的写
- Consistency  the database being in a “good state.“ 其实是应用决定的一致性。
- Isolation  concurrently executing transactions are isolated from each other
- Durability 保证commit成功，数据就不会丢失。(对于单节点应用，意味着写入硬盘，对于分布式应用，可能意味着写入了replica)

### Single-Object and Multi-Object Operations
Multi-object transactions require some way of determining which read and write operations belong to the same transaction。
Single : lock or CAS.

Multi-object 的必要，关系数据库外键， 第二索引， document 数据库的join等

### Handling errors and aborts
retry的缺点：

- 如果transaction成功了，只是返回成功的网络挂了，重试一定不能影响数据  
- 如果失败是因为负荷太大， retry可能会加重问题
- 只对临时的错误有用
- retry 的时候client 挂掉， 会数据丢失

## Weak Isolation Levels
isolation should make your life easier by letting you pretend that no concurrency is happening

真的serializable 性能很差

### Read Committed
最基本的transaction isolation
- 只会读到已经committed的数据    
- 只会overwrite已经committed的数据

如果脏读（未committed的数据），部分写或者回滚时会有问题。
防止脏写，一般是等到另一个transaction提交之后，再写，但是还是存在race condition的问题。

一般通过行锁来实现，写的时候需要获取锁，但是读的时候如果也要获取锁，性能不好，实践中很少用，一般是保存旧的值和未提交的值，读的时候返回旧值，直到commit

### Snapshot Isolation and Repeatable Read
备份的时候，耗时很长， 而且备份过程中一直有新数据写入，如果只有Read Committed， 就会发现备份的数据有些是新版本，有些是旧版本

Analytic queries and integrity checks

Snapshot Isolation， transaction读的数据都是某个时间点的snapshot。写锁互斥，但是不影响读，要实现这个，需要保存不同commit的数据，也叫multiversion concurrency control *(MVCC)*。

每一个transaction有一个id， 每行数据有一个created_by, deleted_by字段，例如在删除的时候，把deleted_by设为transactionId, 等没有transaction访问的时候，gc来删除， update 被 翻译为delete + add。 当一个transaction读数据的时候， 它本身的id会决定读取什么数据， 此时已经在进程中的transaction id 所写的内容不会读， 比自己id大的内容不会读， 标记为delete的不会读取   

read commit 只需要保存两个版本， commit 和 未commit, 一般也是用MVCC来实现的。 

MVCC的index，每一个transaction一个新的root-tree node， 改过的部分新node， 没改的用原来的

### Preventing Lost Updates
两个user都读了一个数据，然后修改，提交，最终会丢失一个人的修改。

- 数据库提供atomic write
- 自动检测丢数据
- 显式Lock
- CAS


### Write Skew and Phantoms
定会议室问题
重名
账户1份钱消费两次

先select， 判断并操作， 写回数据库提交（这个写会影响（另一线程）第一步select的结果， 叫幻读）

## Serializability
regarded as the strongest isolation level. It guarantees
that even though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency

### 真的线性执行
单线程，顺序执行。但是工业应用是07之后的事情，一是因为RAM变得便宜，所有数据放在内存变得可能，transaction少了读写硬盘的时间，短到可以接受。二是因为一般得事务处理数据库，一个事务读写得数据非常有限，而长查询得分析数据库，可以基于snapshot。

redis 就是单线程。

不适合交互多，有很多阶段的事务，比如订机票

#### 存储过程

#### Partitioning
对于写密集的database，单线程很可能成为性能瓶颈，多核没用上，VoltDB支持数据分割，但是跨partition的事务会有额外的控制，性能更差，分割对数据有要求

### Two-Phase Locking
在更长的30年里，两阶段锁是实现线性执行的唯一一个广泛使用的算法。

写排他，读共享。如果B要写，而A在读，B需要等待A commit或者abort。
如果B在写，A要读，也需要等B commit 或者 abort。

与Snapshot isolation 关键的地方就在这里， Snapshot isolation 读不会锁写， 写也不会锁读。

实现：
- 有share 和 exclude两个模式
- 读要获取share锁
- 写要获取exclude 锁
- 先读后再，升级锁

两阶段的意思，第一阶段获取锁执行，第二阶段释放锁。
死锁， 自动检查， 废弃其中一个并重试

#### Predicate locks
没有写的数据也要有锁， select 语句锁

#### Index-range locks
预测锁性能并不好, 粒度太小，范围锁锁得范围比较大

### Serializable Snapshot Isolation (SSI)
基于snapshot 的乐观锁
基于版本读，然后在提交的时候，看是不是还是当前版本，不是就废弃
有人写的时候，不会锁读，对读多的场景很有吸引力
