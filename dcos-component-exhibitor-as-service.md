## Exhibitor on DC/OS

### 准备工作

在DC/OS集群中部署独立的**Exhibitor**服务时，需要提前确认集群中Agent节点的`hostname`是否为节点的IP地址或可解析的域名，而不是**`localhost.localdomain`**。

### 部署

完成上述准备工作后，从`Universe`中选择Exhibitor进行安装，默认创建3个节点的Exhibitor(ZK)服务实例。

也可以通过DC/OS CLI直接安装：

```
dcos package install exhibitor
```

安装完成后，3个ZK节点通过DC/OS集群中的Exhibitor(ZK)服务相互发现，并同步节点状态。可以通过集群的[Exhibitor服务管理界面](http://<MASTER_IP>/exhibitor/exhibitor/v1/ui/index.html)查看基于Zookeeper共享配置的节点信息。共享配置信息默认位于：`/exhibitor-dcos/exhibitor-dcos/configs/config`。

**注意：**

1. 如果在服务安装过程中修改了服务的名称，则位置信息中的`exhibitor-dcos`替换为服务名称。
2. Exhibitor服务部署完成后，如果通过节点管理界面的控制面板查看节点状态，会发现节点相互发现的过程会有“抖动”现象，这是多个节点通过集群的ZK共享节点达成一致引发的。

最终部署完成后配置界面信息类似于：
![](/assets/dcos-exhibitor-service-1.png)

