
## Data Structure
    log-structured (append-only records)
    page-oriented
    index，加速读,写降速
### Hash Index
Bitcask（hash + log)  hash在内存中做索引，value是值得内存byteOffset，适用于频繁更新的数据，读写性能好，因为在内存，key的数量有限制。

一些问题：
- 文件格式。
- 删除记录的时候先添加一条删除记录，在merge的时候在物理删除。
- 数据库重启，内存中的hashMap就没了，需要去读snap-shot
- 一条数据写一半可能数据库崩溃了，需要能检测并废弃这类记录
- 只能单线程写

只能append的原因：
- 线性写性能好（尤其的spinning-disk硬盘, ssd某种程度也是）
- 不用担心update一半，部分数据被更新，部分数据没更新的情况
- merge防止数据是片段式的（如果直接更新，不是merge，删除，update就可能使数据支离破碎）

hash的局限：
- 内存。硬盘hash的性能不好，很多随机IO  
- 范围查询支持不好

### SSTables and LSM-Tree
Sort String Table 将key排序
LSM-Tree : Log Structure Merge-Tree 

- 用mergesort的方法，不会占用太多内存
- 不用将key全部放到内存，隔一段放一个就可以，然后扫描片段
- block,压缩

写读过程：
1. 先写到Tree结构的内存， memtable
2. 当内存到达一定容量，讲排序好的keyValue写到一个SSTable 文件中，新的memtable 实例用于内存写
3. 读先读memtable， 然后最新的SSTable 文件， 然后次新， 然后..
4. 隔段时间merge segment文件， 废物被更新和删除的

有一个缺点就是memtable中的数据可能会丢失， 可以用一个另外的log 来记录写入memtable的记录, 不用排序， 当memtable写入文件的时候，该log就可以废弃了。

LevelDB， RocksDB, Hbase, Cassandra

lucence存储term用了类似的机制，value是包括term的文档id。

LSM的一个缺点是，如果key不存在，搜索时间可能会很长,因为要扫描所有的segment file，有一个解决办法是用Set,将key全部放入，然后先判断key存在与否。

### B-Tree
The most common index

- fix-sized pages（一般是4kb或更大
- 分层, 上层有指向下层的引用，叶子层是连续的数据
- 指向下层的引用数叫branching factor， 一般是几百，一般3~4层
- 当page大小写不下时,会分割成两个一个大小（各一半）的page，在包含key的sub-page上更新，然后再parent里更新两个page的引用。
- 更新数据时,找到page，update数据，然后写回page

当分page的时候，可能page分了，还没有在parent里更新，数据库崩溃,这时候就会出现孤儿page，一般会引入WAL(write-ahead-log)，将B-tree变化的记录写入log。

需要并发控制，加锁.

一些优化：
- 可以只保留部分key，只要在page上能区分就可以
- 将相邻的page写入硬盘相邻位置（但是tree变大变化时维护也变难了
- 同一层之间的page，也可以有互相引用的指针

### Comparing B-Trees and LSM-Trees
粗略地讲, LSM-Trees写更快，B-Tree读更快

B-tree不好，LSM-Tree好
- B-tree至少写两次，一次WAL，一次Page，更改page即使只改一点，也会重写page，重写也可能写两次
- LSM-Tree更紧凑
- LSM-Tree和硬盘写的方式更搭

LSM-Tree不好
- LSM-Tree因为merge，一条数据可能被写很多很多次，对SSD不友好
- compaction,Merge expensive， 查询方差大
- compaction的速度可能跟不上新数据的速度（一般不会

### Other Indexing Structures
- heap file (存储文件，索引指向这里)
- clustered index （数据跟索引存储在一起
- index cover query (查询的数据都在索引里)

#### R-tree
搜索地理位置，经度范围,维度范围

#### Full-text Search and Fuzzy index

### In memory DataBase

## Transaction Processing or Analytics

事务处理和数据分析的需求不同。汇总大量记录, 批量导入，历史数据

### Data warehouse
不影响业务数据，只读,批量导入，分析友好
ETL Extract-Transform-Load

### Stars and Snowflakes: Schemas for Analytics
中间有一个fact-table(记录事件)， who, what, where, when, how, and why ，
表中的数据可能引向别的表(dimension table)，一般会非常大，超过100行，周围的表也非常多

### Column-Oriented Storage
fact-table 列很多，但是用的时候经常只会用很少的列，关系数据库或者document数据库会把所有行数据读到，very expensive   

#### column compression
 列数据一般重复性特别高

bitmap encoding, n distinct values and turn it into n separate bitmaps。 
当n很小时,直接存储bitmap对应的值, 当n很大的时,n中会有很多0，可以依次记录0的个数，1的个数。
查询的时候变为相应bitmap的查询  

对处理器很友好，bitmap的处理不用调用方法，没有分支预测,and or的操作可以向量化运行

根据某列排序(整行跟着排序),这是可能会出现大量重复的值在一个区间,描述开始和长度,可以进行非常大的压缩(首行)

### Writing to Column-Oriented Storage
因为有column压缩,插入更新数据expensive，用类似LSM-Tree的方法，先写到memtable，
积累要一定量后再merge写进硬盘，查询的时候查询memtable和disk，将结果组合起来。