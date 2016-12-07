## 部署Storm集群

在DC/OS中部署Storm集群环境，主要是指部署Storm的 **Nimbus**、**UI**和**DRPC**服务。Storm的Supervisor/Worker服务是在向Storm集群部署拓扑（Topology）时，由Mesos自动分配资源调度执行。

### Nimbus部署

在本实现的镜像中，通过Marathon部署Nimbus时，Storm的UI服务可以在启动Nimbus时默认同时启动。

根据前述Storm集群配置所述，即使在storm.yaml中配置了不同于mesos.master.url的storm.zookeeper.servers配置，服务启动时默认仍使用mesos.master.url中配置的ZK集群信息，因此，在DC/OS中部署Storm集群时，根据是否使用DC/OS默认ZK集群，Marathon的应用配置稍有差别。

#### 使用DC/OS的Exhibitor(ZK)服务

在使用DC/OS默认的Exhibitor(ZK)集群服务时，mesos.master.url通常配置为：zk://mesos.master:2181/mesos。storm.zookeeper.servers配置为：mesos.master。即：

```
mesos.master.url: zk://mesos.master:2181/mesos
storm.zookeeper.servers:
 - "mesos.master"
```

Storm集群的配置信息默认存储在ZK的/storm节点下。

Storm集群默认配置数据目录存放在“storm-local”下，相对于/opt/storm路径。可以通过容器卷挂载将数据存放到持久化卷或外部存储，如NFS。

可以通过容器卷挂载，将/opt/storm/conf和/opt/storm/log4j2挂载到外部存储，让Storm集群能够读取外部配置。

可以

此种方式的

#### 使用独立的Exhibitor(ZK)服务

### DRPC部署
