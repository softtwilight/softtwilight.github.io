> 语言的边界就是我世界的边界        -- 维特根斯坦

## Relational Model
    hide that implementation detail behind a cleaner interface.

## Nosql
    open source, distributed, nonrelational databases.
    Not only Sql.
## The Object-Relational Mismatch
    Object-orient language
    Relational Model
    mistch between objects and tables, rows, columns.
    ORM : Object-relational mapping 
## Many-to-One and Many-to-Many Relationships
    例如地区数据，通过下拉列表选择,可以保持一致性，容易更新，易于搜索。存储地区id，可以减少数据重复 
    normalizing (many to one) 许多人住在一个地区
    many-to-many relationships 当年的hierarchical model很难解决的问题

### The network model  
        query is access paths,（最小路劲算法？）,查询很难,很不灵活，需求变更也会哭的

### The relational model
        查询的复杂度隐藏在内部，general-purpose query optimizer
        join 支持很好
### document databases 
        1对n的时候，把n作为子节点存储;n对n或n对1时,如关系型数据库的外键,document reference引向别记录.
        因为数据的locality，一般搜索性能比关系型数据库好。
        数据一般更接近应用的object， schema更灵活
        For highly interconnected data， 有些尴尬

## Query Languages for Data

###  query language
- concise and easy
- hides implementation details of the database engine
- easy to parallel execution.

### MapReduce Querying
    介于命令式和声明式之间
    must be pure functions

## Graph-Like Data Models
    many-to-many relationships are very common.
    - vertices (nodes)
    - edges (relationships)

### Property Graph
点vertex有以下性质：
- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)
边edge有以下性质：
- A unique identifier
- The vertex at which the edge starts
- The vertex at which the edge ends
- A label to describe the kind of relationship between the two vertices
- A collection of properties
非常灵活

#### The Cypher Query Language
输入vertices and edges 后可以搜很多非常有意思的事情
比如搜索从欧洲移民到美国的人的名字：

`MATCH
(person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name`
WITHIN*0 表示0或更多withIn，这种不确定的搜索关系，在关系数据库很难表达

### Triple-Stores and SPARQL
所有的数据store as (subject, predicate, object)， example (Jim, likes, bananas)
predicate　可能是一个edge，也可能代表property

#### The SPARQL query language
`SELECT ?personName WHERE {
 ?person :name ?personName.
 ?person :bornIn / :within* / :name "United States".
 ?person :livesIn / :within* / :name "Europe".
}
`
#### Datalog
写rule



