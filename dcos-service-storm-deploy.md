## 部署Storm集群

在DC/OS中部署Storm集群环境，主要是指部署Storm的 **Nimbus**、**UI**和**DRPC**服务。Storm的Supervisor服务是在向Storm集群部署拓扑（Topology）时，由Mesos自动分配资源执行。

### Nimbus部署

在本实现的镜像中，通过Marathon部署Nimbus时，Storm的UI服务在启动Nimbus时默认同时启动。

#### 使用DC/OS的Exhibitor(ZK)服务

#### 使用独立的Exhibitor(ZK)服务

### DRPC部署
