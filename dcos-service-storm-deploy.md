## 编译与部署

### 编译

从[Github仓库](http://github.com/christtrc/storm)下载源代码。

### 部署

在DC/OS中部署Storm集群环境，主要是指部署Storm的 **Nimbus**、**UI**和**DRPC**服务。Storm的Supervisor服务是在向Storm集群部署拓扑（Topology）时，由Mesos自动分配资源执行。

在本实现的镜像中，Storm的UI服务在启动Nimbus时默认同时启动。



