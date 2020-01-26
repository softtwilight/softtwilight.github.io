Partitioning and Replication　经常一起使用

How to partition?
## Partitioning of Key-Value Data
尽量让读写平均到各个节点上

### Partitioning by Key Range
- 怎么选择选择分的范围
- 数据很集中会出现hot spot

### Partitioning by Hash of Key
- 分的很均匀
- 不能范围查询
- 极端情况，一个key有非常多的读写，也不能避免hot spot(明显出轨...),可以再key上加上两位随机数（就可以产生100个不同的key），有个缺点就是，读key的时候，可能就需要读100个不同的key然后汇总，然后这种方案只适用于大量读写的情况，所以在什么时候加也是一个问题。 

### Partitioning Secondary Indexes by Document
各个分片上还有自己的索引(local index), 比如汽车id是用来分片的index， 分片上还有颜色的索引。搜索颜色的时候，就需要把请求发送到所有分片，然后收集起来。
scatter/gather， tail latency amplification 木桶效应 

## Rebalancing Partitions
读流量变大，dataset数据量变大，节点挂掉

平衡，继续接受读写，尽量少的传输数据

### Strategies for Rebalancing
- 如果是hash， 最好用范围hash to partition, 移动数据少，灵活
- Fixed number of partitions 分为很多， 比如20个， 4个节点， 每个节点5个partition， 然后扩充为5节点，每个节点4个partition(每个节点移动一个partition到新节点)， ES， Riak就是用的这个， 决定每个节点多少partition是一个问题。
- Dynamic partitioning, key range–partitioned databases, 当一个partition的size 到达一个值后， 就再分割成大小一样的两个partition
- a fixed number of partitions per node

## Request Routing
service discovery
- client 可以访问任意节点， 节点如果不含有查询的数据， 转发请求
- client 先请求到一个路由层
- client 意识到patition和在node的分布，可以直接请求响应节点