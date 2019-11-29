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
