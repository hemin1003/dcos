## DC\/OS概览

DC\/OS即数据中心操作系统，是一个架构于现有单机操作系统之上的统一调度管理一个规模硬件集群的CPU、内存、磁盘和网络等资源为上层应用提供抽象的资源分配调度的分布式操作系统。

![](/assets/dcos_overview.png)

可以借用传统操作系统的内核空间与用户空间两个概念来理解DC\/OS。如上图所示，最底层是一组主机操作系统网络组成的规模硬件集群；中间层的DC\/OS内核空间是一个分布式框架，该框架归集底层规模主机集群的所有CPU、内存、磁盘和网络等资源，并为上层用户空间中的用户应用及服务提供资源分配、安全和进程隔离；最上层的DC\/OS用户空间包括一组基础系统服务，用户应用服务和用户应用服务管理的进程。

下面来进一步理解DC\/OS的内核和用户空间：

![](/assets/dcos-architecture-100000ft.png)

DC\/OS的内核空间

如上图所示，DC\/OS的内核空间包括Mesos Master和Mesos Agent两种类型的服务，分别部署在独立的主机操作系统上。

根据集群硬件规模及服务可靠性要求，通常Mesos Master节点的数量为1、3、5个（其中一个Master为Leader节点，通过Zookeeper进行选举），在硬件资源有限的环境中，如本地环境，1个Master节点即可。在生产环境中，Master节点保持3个或5个。

注意，并不是Master节点的数量越多越好\[LINK\]，对于超大规模硬件集群，5个Master节点完全足够。当一个Leader Master节点崩溃或下线，一个新的Leader Master节点会被自动选举出而不影响当前正在运行的服务。

Mesos Agent部署在Master节点之外的所有主机上，收集这些主机的CPU、内存、磁盘、网络等资源，提交给Master节点。因此，Mesos Master可以掌控整个集群中所有可供分配的资源。

Mesos Agent还负责管理运行在Agent节点上的业务服务，这些服务可运行于两种容器中：Mesosphere容器和Docker容器。

DC\/OS的用户空间

DC\/OS的用户空间除了用户服务之外还包括一组系统组件服务（30多个）负责，根据Master节点和Agent节点的不同，这些系统组件服务有所不同。这些服务在DC\/OS集群创建时安装并运行，下面简单列举几个：

* **Admin Router** 通过一个Nginx配置提供安全检查并代理DC\/OS服务。

* **Exhibitor** 在安装时自动配置Zookeeper，并为安装的Zookeeper提供一个Web UI。

* **Mesos-DNS** 提供一种服务发现机制，可以让应用和服务通过DNS互相发现。

* **Minuteman** 是一个内部的4层负载均衡实现。

* **Distributed DNS Proxy **一个内部的DNS调度实现。

* **DC\/OS Marathon** 原生的Marathon实例，用于承担DC\/OS系统的初始化，并提供DC\/OS服务的启动与监控。

* **ZooKeeper** 高性能的分布式协调服务。


DC\/OS的系统组件服务将在后续章节详细介绍。

DC\/OS的用户空间主要用来运行用户服务，这些服务可以是遵循DC\/OS服务规范的任意的应用或Docker容器镜像。常见的服务包括：Marathon，Cassandra，Kafka，

DC\/OS的启动过程

