## 服务发现与负载

要理解DCOS的服务发现与负载均衡策略，首先要理解DCOS的分层和区域划分。但是，DCOS随着集群及组网规模的不同，业务场景的不同，其分层和区域划分策略也不相同，进而影响服务发现与负载均衡的选型策略，在实际应用中应根据具体需求做出决策。

在阐述DCOS为服务发现与负载均衡所提供的各种实现之前，我们先想象一下下述场景：

* 应用集群都是内部的TCP服务调用负载均衡，不需要Layer 7的特性。

* 应用集群需要对外提供TCP和（或）HTTP服务调用。

* 应用集群需要Layer 7的负载均衡特性，如：TLS Termination，zero-downtime deployments， HTTP sticky sessions，load balancing algorithm customization，network ACLs，HTTP basic auth，gzip compression等。

* 应用集群的其它服务发现与负载均衡需求。

针对上述场景，DCOS提供了VIPs（Minuteman），Marathon-LB，Mesos-DNS等服务组件及策略。

* VIPs（Minuteman）是一个4层负载均衡实现，用于DCOS集群内部Mesos任务之间大多数TCP流量的负载均衡。

* Marathon-LB（MLB）是一个基于HAProxy实现的，为Marathon提供轻量级的TCP\/HTTP路由和负载均衡的服务。

* Mesos-DNS是一个简单的基于DNS的服务发现工具，可用于Mesos任务之间服务发现。


为了覆盖DCOS为服务发现与负载均衡所提供的各种实现，我们以一个典型的服务拓扑为例进行阐述。

【图】

These E-W\(East-West\) VIPs are handled by two internal pieces of software called minuteman \(load balancer\) and spartan \(distributed DNS proxy\). They're way faster to propagate information than Mesos-DNS so your app availability will be much improved.

