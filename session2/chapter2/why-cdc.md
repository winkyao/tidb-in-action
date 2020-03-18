# TiCDC 解决什么问题

### 简述：
TiCDC （Change Data Capture） 是用来识别、捕捉、交付 TiDB/TiKV 上数据变更的工具系统，可以作为 TiDB 增量数据同步工具，将 TiDB 集群的增量数据同步至下游数据库或生成 open TiCDC protocol 数据发布到第三方系统，包括消息队列，文件存储系统等。TiDB 4.0 引入，相比于之前版本的 TiDB-Binlog 工具，不再依赖 TiDB 事务模型保证数据同步的一致性，系统可水平扩展且天然提供高可用的特性。

### 适用场景：
+ 流式订阅
 	+ 构建分析平台 Hive，Spark 等
	+ 构建全文搜索 Elasticsearch 等
	+ 通知事件-更新缓存，共享状态变更
+ 数据备份
	+ 同构数据库-复制数据构建灾备集群
	+ 异构数据库-复制数据构建灾备集群
	+ 增量备份-恢复到任意时间点

### 为何要引入 TiCDC：
TiDB-Binlog 在之前使用的过程中，有比较多的限制和问题，举例说明：
+ 易用性差：至少需要部署 pump 、drainer 组件，排查问题困难。
+ pump 的 relay log 单点，如果某个 pump 节点的 binlog 丢失，会造成同步无法继续，必须重做下游数据，重新搭建 TiDB-Binlog 同步。
+ drainer 单点归并排序，处理效率不高，无法应对超大规模 TiDB 集群。
+ 开启 binlog 的情况下，事务层需要保证同步的完成 binlog 写入，事务才算提交成功。TiDB-Binlog 异常情况下，也会影响上游 TiDB 集群的事务正常提交。
+ TiDB-Binlog 需要解决数据安全和服务高可用问题，目前都未实现。
+ TiDB-Binlog 极端情况下(丢失 c-binlog)，需要反查 TiKV 事务状态，同步延迟会增加。
使用原有的 TiDB-Binlog 架构已经无法解决这些问题，基于这个考虑，我们设计和开发了 TiCDC。
TiCDC 有以下优点：
+ 对上游 TiDB 集群影响小: 不再依赖于 TiDB transaction 模型，不会因为同步工具异常影响到上游 TiDB 集群的正常使用。
+ 扩展性好: TiCDC 可以优雅的 scale to any cluster size，这一点 TiDB-Binlog 要弱于 TiCDC，虽然 pump 集群具有一定的扩展性，但是 drainer 是单节点归并排序，无法应对超大规模 TiDB 集群，TiCDC 提供了更良好的扩展性，可以应对超大规模 TiDB 集群的使用场景。
+ 延迟低: TiCDC 直接监听 TiKV 层数据变更，TiCDC 内部同步推进模型支持多个 TiCDC 进程并发写入下游，保证了 ms 级别低延迟。
+ 高可用：上游 TiKV 可以保证对 TiCDC 提供的增量数据高可靠，TiCDC 各节点无需存储额外数据。TiCDC 跟 TiDB 一样无状态，不存储数据，通过 PD 来协调任务调度，保存同步任务、同步状态等元信息，动态增删 TiCDC 节点不会影响同步继续往下同步，但是会在处理过程中增加一些延迟。
+ 易用性好: TiCDC 是 all in one binary，部署简洁。
+ 可用性好: TiCDC 可以全部使用 SQL 管理，不需要另外组件，自带 admin ui。
