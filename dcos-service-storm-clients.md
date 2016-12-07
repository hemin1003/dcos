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

3. 如果拓扑未能成功部署，可通过http://<DCOS_MASTER>/mesos进入Mesos管理界面，查看已完成的任务，找到拓扑对应的任务，进入到sandbox，查看日志信息。