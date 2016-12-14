## 访问Storm集群

向Storm/Mesos集群部署拓扑的方式与向独立的Storm集群部署拓扑的方式完全相同。

首先确定Nimbus服务在DC/OS集群中的位置和服务端口。这可以通过DC/OS WEB管理界面查看。

一种部署拓扑的方案如下：下载Storm安装包到本地并解压，在控制台进入到解压目录，执行下述命令（示例）：

```
/bin/storm jar -c nimbus.host=192.168.1.81 -c nimbus.thrift.port=14556 examples/storm-starter/storm-starter-topologies-1.0.2.jar org.apache.storm.starter.RollingTopWords word-count-2 remote
```

然后通过Storm UI确认拓扑部署的情况。

如果拓扑正常部署但启动出现问题，排查顺序是：

1. 首先通过Storm UI检查拓扑的部署信息，确认拓扑是否正常部署。因拓扑部署时，框架需要获取资源，并动态构建Supervisor及Worker，因此通过Storm UI观察时，可能需要等待并及时刷新界面。

2. 如果拓扑部署成功，通过Spout/bolt信息确认Worker动态分配的Agent节点，如果能够通过Storm UI查看日志，则直接查看。否则，进入到DC/OS管理界面，通过Node进入到节点列表，然后选择拓扑部署的节点，进入后查看拓扑服务的日志信息。

3. 如果拓扑未能成功部署，可通过`http://<DCOS_MASTER>/mesos`进入Mesos管理界面，查看已完成的任务，找到拓扑对应的任务，进入到sandbox，查看日志信息。

### 动态调度的差异

前文讲过，Storm的Nimbus部署拓扑的过程实际上是为Executor（包含了拓扑部署信息）选定具体的Host+Port（Slot）映射，也就是构建WorkerSlot的过程。在传统的Storm集群中部署拓扑时，由于可用资源是固定的（即通过Supervisor构建的资源池是静态的），Nimbus只需选定Supervisor节点以及节点上可用的Slot即可。 

在Storm/Mesos集群中，资源不被Storm集群独占，仅在拓扑部署时，根据拓扑需要的资源，Storm调度器从Mesos提供的资源池中调配出需要的资源，并将Executor传递到选定的这些资源的节点上，这个资源调配过程需要额外的时间。因此，通常需要调整Nimbus的两个配置参数加大相应的时间配置，否则可能出现类似：`Executor example_top:[1 1] not alive`的异常。

* nimbus.task.launch.secs: Executor在调度节点上的启动时间，默认值为120秒。

* nimbus.task.timeout.secs：Nimbus检查Executor是否正常的心跳超时时间，默认值为30秒。