Replication means keeping a copy of the same data on multiple machines.

优点：
- 物理上可以让数据更靠近用户
- 部分机器down掉还能正常运行
- 机器可以根据请求水平扩展

难点在于数据改变了，怎么处理

### Leaders and Followers
怎么保证数据写到每一个replica上？
一种最常见的方式是leader-based replication.
It works as follows:

1. 其中一个replica被任命为leader(master/primary),写数据时先发请求到leader，leader写到local storage。
2. 其他的replicas被称作followers。leader写到local storage后，会发送data log到followes, 以相同的数据顺序执行写操作
3. 读数据可以访问任何一个replica

### Synchronous VS Asynchronous Replication
- synchronous: leader 发送消息到follower，得到写成功的response再返回给用户。如果某一个节点出问题，就得不到返回。

- Asynchronous: leader不确认follower写成功就返回。

- semi-synchronous: 只有一个节点时同步的(leader 挂掉也不会丢数据)

一般配置为完全异步的(特别是多followers, 跨地域时)

### Setting Up New Followers
1. 对leader的database take a snapshot, 尽量不要锁表
2. 把snapshot 复制到新的follower
3. follower 连接到leader， 请求自snapshot点后所有的change data
4. follower 赶上leader，继续接受新的change data

### Handling Node Outages
#### follower 挂掉
在每一个节点都有一个log,记录从leader收到的data change， 重启之后， 可以从日志中知道最后一条处理的记录，向leader请求自这条记录之后的数据

#### leader 挂掉
- 确认leader 已经挂掉， 一般通过发送心跳
- 选择一个新的leader
- 重新配置系统使用新的leader。旧的leader恢复后，要让它变成follower

可能的问题：
- 如果新选取的leader是Asynchronous的节点，那么写入旧leader的数据，在新leader上可能不存在，会产生冲突，比较常用的方法是废弃掉旧leader上没有写入其他replica的数据，但是就丢数据了。
- 有时候丢数据还有别的问题，比如采用自增主键的数据库，新旧leader的count不一样。
- 选取机制设计不好，可能会多leader，写冲突
- 确认leader挂掉的时间选择，长了会让recovery更久，短了可能会引起没必要的更换leader

trade-offs around replica **consistency, durability, availability, and latency** are in fact fundamental problems in distributed systems. 

### Implementation of Replication Logs

#### Statement-based replication
数据能直接传sql到replica吗？
如果有now(), rand()函数呢， mysql 5.1之后用基于行的replication

#### log-based replication
log的缺点是，log是非常低层次的数据描述，如果数据库更换引擎或者升级，有可能就不能rolling 升级了(先将follower换成新版本，然后挺掉leader，升级后再上线)

#### Logical (row-based) log replication
真实的数据，MySQL’s binlog

#### Replication 的缺点

单leader模式，写都通过leader， 所以适用于读多写少的场景， 读多写少(read-scaling architecture)的场景， follower多， 用同步写几乎是不现实的， 异步写又会出现读的数据不同步的情况, 这中不一致是个暂时的情况， 如果停止写并等待一定时间， 状态最终会一致的， 这就是所谓的 **最终一致性**。

如果leader写和follower写的间隔很长，比如提交了一个评论，显示提交成功，但是刷新页面， 又没看见评论。**read-your-writes consistency**

- 直接read from leader（内人主页）
- 客户端记录写的逻辑时间，如果replica没有写在这个时间之后的内容，从leader读（跨设备访问的uese不可行

Monotonic read： 先读到了内容，刷新，被路由到另一个replica， 这个replica的延迟大，又没有内容了，very confused, 可以让一个用户始终在一个replica上读

后写的可能被先读到

### Multi-Leader Replication
 每一个leader同时也是其他leader的follower。
 对于单个datacenter，一般很少用，因为复杂度大大提高

 多数据中心：
 数据中心内同步写，数据中心之间异步写，性能，容错更好，但是不同的数据中心可能同时修改一份数据，冲突解决。。避免：一个特定的文件只能一个leader写。

 还可能出现写的顺序不一致，在a节点是x->y, 在b节点是y -> x;

 其他方案：
 - last write win
 - 记录下两次写，之后解决...

 ### Leaderless Replication
写到每一个replica,多数写成功就ok，读的时候也在多节点读，然后根据版本号对比取到的值，返回最新的（也可能在这里更新旧数据， 也可能是后台一个寻找差异的进程来更新）

需要写成功的replica数和读取的多节点数加起来要大于总的replica数，可以保证一定会读取到新数据。

但是实际运用上还是会有可能读到旧数据。

根据key记录一个version num， 每次写会更新num, 读会把num一起返回， 提交write的时候，version num以前的数据被覆盖掉， 之后的merge在一起，然后返回最新的数据（merge 后的）