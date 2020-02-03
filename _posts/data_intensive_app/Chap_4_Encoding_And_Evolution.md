需求变化的时候数据结构一般也会变化
rolling 发布， 向前兼容，向后兼容

## Formats for Encoding Data
一般应用的两种数据呈现：
- data in memory
- data to a file or network

### Language-Specific Formats
类似java.io.Serializable， 直接将内存数据输出：
- 读这类数据比较困难，语言限定
- 安全问题
- 数据向前向后兼容一般被忽视
- 性能差

### JSON, XML, and Binary Variants

文本式的JSON，XML，CSV的一些缺点：
- XML，CSV不能区分数字和数字形式的字符串，JSON不能保留小数精度
- JSON and XML 不支持 bianary string, 替代方案是用base64编码
- JSON and XML 有内在的schema，读数据会依赖schema，应用不用对应schema的要hardcode，比较复杂
- CSV格式靠约定，比如加一列可能就不能读了

#### Binary Encoding

二进制的JSON，XML，牺牲了可读性，但是压缩提升很小

### Thrift and Protocol Buffers
需要对数据进行定义:

`struct Person {
 1: required string userName,
 2: optional i64 favoriteNumber,
 3: optional list<string> interests
}`

将类型(str,num,list)和前面的序号(tag)一起编码为一个数字 + 长度 + 数据.

当数据定义变化时，新增或删除tag(不能是required)，程序会忽视自己没有定义的tag

### Avro

`record Person {
    string userName;
    union { null, long } favoriteNumber = null;
    array<string> interests;
}`
encode 没有tag(序号),所以必须依次执行,encoding 和 decoding的schema一定要一样

有write schema 和 read schema,并且有一个机制让read schema 适配 write schema, 读写schema都有自己的版本号，可以用database来保存这些version。

移除tag的一个非常好的地方在于，可以动态生成schema文件了，所以数据结构变更的时候不用人工介入。

Avro, Thrift， Protocol Buffers的schema都比xml，json的schema简单

binary encoding的一些好处：
- 体积小，去掉name的部分
- schema本身有文档的价值
- 对于静态语言，可以通过schema来生成代码，编译检查
- 数据库保存schema version， 在发布前可以验证兼容性


## Modes of Dataflow

### Dataflow Through Databases

### Dataflow Through Services: REST and RPC
clients/servers

#### web services  
- **REST** 格式简单，用url定位资源，http特性来认证，cache，内容negotiation。
- **SOAP** an XML-based protocol, 能够生成代码，让本地调用远程服务向调用本地方法一样(框架先encode为xml格式，然后再decode)，不太适合动态语言  

location transparency

RPC的缺点：
- local func call是稳定的，远程调用未知，无法在自己的控制下
- 可能不会返回结果
- 如果重试，有可能第一次调用就成功了，只是返回丢失了
- 耗时
- 调用本地方法可以传引用，RPC需要把对象encode
- RPC跨语言会很ugly

### Message-Passing Dataflow

相较于RPC的优点：
- 当接受方服务不可用的时候，可以作为buffer，提高reliability。
- 可以自动重发，保证不丢失message
- 不用意识到接受放的ip和端口（在云环境很有用）
- 分发
- 解耦

不过一般是单向的
不规定encode

#### Distributed actor frameworks

Rolling Upgrade, 保证向前向后的兼容性