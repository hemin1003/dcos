## Mesos-DNS

Mesos-DNS为DC\/OS集群中的应用提供了服务发现功能，可以让DC\/OS上运行的应用程序和服务通过域名系统（DNS）找到彼此。例如，由Marathon或Aurora启动的应用程序可以分配名称：search.marathon.mesos或log-aggregator.aurora.mesos，Mesos-DNS将这些名称转换为当前运行各个应用程序的主机上的IP地址和端口。要连接到DC\/OS中的应用程序，必须要知道的应用的名称，每次启动连接时，DNS转换都会将这些连接指向数据中心中正确的主机。

Mesos-DNS被设计为一个精简的，无状态的，易于部署和维护的服务。下图描述了它的工作机理：

![](/assets/mesos-dns-architecture-dcos.png)

Mesos-DNS定期（默认30秒）查询Master节点，从所有正在运行的框架中检索所有正在运行的任务的状态，并为这些任务生成DNS记录（A和SRV记录）。随着任务在DC\/OS集群上启动，完成，失败或重新启动时，Mesos-DNS会同时更新DNS记录以反映应用的最新状态。

框架不需要直接与Mesos-DNS进行通信。在Agent节点上运行的应用程序和服务可以通过发出DNS查找请求或通过REST API发出HTTP请求来发现其所依赖的其他应用程序的IP地址和端口，Mesos-DNS会直接回复这些请求。如果Agent节点需要对DC\/OS群集外部的主机名发出DNS请求，Mesos-DNS会直接查询外部DNS服务器。仅当DC\/OS群集上的节点必须解析集群网络之外或Internet上其他位置的系统的主机名时，才需要外部DNS服务器。

Mesos-DNS是简单和无状态的，它不需要一致性机制，持久存储或日志复制，因为应用所需的心跳，健康监控或生命周期管理等功能都已经由Master、Agent和框架所提供。Mesos-DNS可以通过使用像Marathon这样的框架来实现容错，可以监控应用程序运行状况并在应用出现故障时重新启动它。Mesos-DNS因故障重新启动时，无需进一步协调它会从Master节点检索最新的应用状态，然后处理DNS请求。可以通过增加Master节点，即可提高可用性并为大规模集群中的DNS请求提供负载均衡。

当应用通过多个框架加载，而不仅仅是Marathon，Mesos-DNS是非常有用的。每个容器都有一个IP，类似[Project Calico](https://www.projectcalico.org/)的解决方案。

查看所有服务

http:\/\/my-cluster\/mesos\_dns\/v1\/enumerate

