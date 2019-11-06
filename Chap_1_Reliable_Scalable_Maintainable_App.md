## Reliability

持续的正确运行：

- 功能正常
- 能容忍用户错误, 非正常使用
- 正常的负载和数据容量情况下，性能足够好
- 无权限用户不能访问

### 容错
- 硬件错误（相对独立）
    - 硬件冗余
    - software fault-tolerance (集群)
- 软件错误
    - 程序bug
    - 占用所有资源（cpu， 内存， 硬盘空间， 网络带宽）
    - 延迟导致无响应, 或者数据陈旧
    - 级联错误, small fault -> big fault
    - solutions：
        - carefully thinking about **assumptions and interactions** in the system
        - 测试
        - process isolation
        - 允许程序崩溃并重启
        - 测量，监控，分析运行情况
- 人为错误
    - 减小错误机会的系统设计
    - 发生错误的地方与运行错误的地方解耦。增加沙盒测试
    - 单元测试,系统集成测试,人工测试,自动测试
    - 能够快速容易的recovery
    - 监控
    - 好的管理


## Scalability
A system’s ability to cope with increased load.

### Describing Load
推特12000发布每秒。写load
    写到中心tweets库 
    or 
    写到不同follower的cache
    or both
刷新首页30万次每秒。读load
### Describing Performance
当load增加：
    a. 资源不变的情况下，性能影响多少
    b. 性能不变的情况下，需要增加多少资源
指标：
- 吞吐量
- 响应时间
    - 平均值
    - 中位数
    - 方差
    - P95 P99 P999 (High percentiles of response times)
    - 原因：
        - TCP丢包，再送
        - GC暂停
        - 需要强行读取硬盘

水平扩展和垂直扩展
弹性系统和人工扩容
stateless service 和 stateful service

## Maintainability

design principles for software systems:

### Operability 运维团队更容易操作
- 提高运行情况的可见性
- 提供自动化和集成支持
- 不要依赖单独的特定的机器
- 好的文档和操作手册
- 提供好的默认行为,也要给管理员更改默认行为的可能
- 合适的时候自我治愈，也允许管理员手动操作
- 展示可预测的行为,最小化意外
### Simplicity 新的开发能容易理解系统
一些不好的：
- 暴露公共状态
- 模块耦合
- 纠缠的依赖
- 不一致的命名，术语
- 解决性能问题的hack
- 随处可见的特殊情况

减少 accidental complexity: 不是要解决的问题，而是实现带来的复杂性。
解决复杂性最好的一个工具是抽象。
### Evolvability 容易开发新功能
