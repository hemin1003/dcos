## Exhibitor服务（Zookeeper监控）


### 特性

Exhibitor主要包括以下特性/功能：

* 实例监控

  Exhibitor实例监控在同一服务器上运行的ZooKeeper服务器。如果ZK没有运行，Exhibitor会写入zoo.cfg文件（请参阅下面的ZK集群配置）并启动它。如果ZooKeeper由于某种原因崩溃，Exhibitor也会重新启动它。

* 日志清理

  在ZooKeeper 3.4.x之前的版本中，日志文件需要维护，Exhibitor会负责定期维护。

* 备份/还原

  ZooKeeper集群中的备份比传统数据存储（例如RDBMS）更复杂。一般来说，ZooKeeper中的大部分数据是短暂的。盲目恢复整个ZooKeeper数据集可能会造成更大危害，因此，需要选择性的恢复以防止对数据集的子集造成意外损坏。Exhibitor提供了这一功能。

  Exhibitor会定期备份ZooKeeper的事务文件，备份后，就可以对这些事务文件建立索引。

* 集群配置

  Exhibitor为整个Zookeeper集群提供了一个独立的控制台，通过它所做的配置更改会对整个集群有效。以下是一些共享配置值：

|Name|	Description|
|--|--|
|ZooKeeper Install Dir|	Path to the ZooKeeper server installation|
|ZooKeeper Data Dir|	Path where ZooKeeper should store its data|
|Log Index Dir|	Path where indexed transaction logs should be kept|
|Servers	|List of servers/server-ids in the ensemble|
|Additional Config	|Additional fields/values to store in zoo.cfg|

* 集群滚动升级

  Exhibitor可以以滚动方式更新集群中的服务器，以便在进行更改时让ZooKeeper集群确保Quorum设定的最低服务能力。

* 自动实例管理

  Exhibitor可以配置为自动向集群中添加新实例，并删除陈旧的实例。这使得ZooKeeper集群可以实现“无接触交钥匙管理”。

* 可视化

  Exhibitor为ZooKeeper提供了ZNode层次结构的图形树视图。

* ZK数据维护

  启用后，维护人员可以在ZooKeeper的存储层次结构中创建/更新/删除节点。

* 死锁检测

  当使用Curator的锁方案（或类似）时，Exhibitor可以分析一组表示锁的ZNode，并确定是否存在潜在的死锁。

* Curator集成

  Exhibitor和Curator可以集成工作，当集群中的信息变更时，Curator实例可以同步更新。

* REST API

  Exhibitor提供了一组用于程序集成的REST API。


### 参考

https://github.com/dcos/exhibitor

