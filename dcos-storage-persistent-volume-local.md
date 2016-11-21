## 本地持久化卷

通常情况下，容器应用在停止并重启后会丢失状态。实际应用中，一些应用如MySQL等希望保持其状态，为了满足这种需求，其中的一种方案是采用本地持久化卷。

如果为应用指定了一个或多个本地持久化卷，任务及其相关数据被“固定”到它们首次启动的节点，在任务停止并重启后仍在该节点运行。同时，应用程序需要的资源也会得以保留。Marathon将隐式保留适当数量的磁盘空间（通过`persistent.size`在卷中声明的）以及为应用程序初始定义沙箱磁盘空间作为应用运行总的磁盘空间。

### 采用本地持久化卷的优势

* 动态保留运行状态服务任务所需的所有资源，从而确保在需要时使用相同的卷在同一节点上重新启动任务的能力

* 不需要限制将任务固定到其数据所在的特定代理

* 仍然可以使用约束来指定分发逻辑

* Marathon允许您定位和销毁未使用的持久性卷，如果您不再需要它


### 创建本地持久化卷

先决条件

参考“安装环境准备”。

配置选项

```
{ 
    "containerPath": "data", 
    "mode": "RW", 
    "persistent": { "size": 10 }
}
```

**containerPath：**应用读取和写入数据的路径。它必须是相对于容器的单级路径;它不能包含正斜杠（\/）（可以是“`data`”，但不能是“`/data`”，“`/var/ data`”或“`var/data`”）。如果应用程序需要绝对路径或带斜杠的相对路径，请使用此配置。

**mode： **卷的访问模式。目前，“RW”是唯一可选的值，可以让应用从卷读取和写入卷。

**persistent.size：**持久卷的大小（以MB为单位）。

需要在APP的JSON定义中设置“`residency`”节点以便告诉Marathon设置有状态应用程序。目前，唯一有效的选项是：

```
"residency": { "taskLostBehavior": "WAIT_FOREVER" }
```

设置不受支持的容器路径

`containerPath`的值必须是相对的，以便于向运行的容器动态添加本地持久性卷，并确保跨操作系统的一致性。但是，应用程序有时可能需要绝对路径或容器路径，或含有斜线（\/）的相对路径。

如果应用程序需要上述不受支持的`containerPath`，则可以通过配置两个卷的方式实现。首先，第一个卷具有所需的绝对容器路径，并且没有`persistent`参数，而且第一个卷的hostPath参数将与第二个卷的containerPath值所定义的相对路径相匹配。如下所示：

```
"volumes": [
    { "containerPath": "/var/lib/data", "hostPath": "mydata", "mode": "RW" }
    { "containerPath": "mydata", "mode": "RW", "persistent": { "size": 1000 } }
]
```

### 参考

https:\/\/mesosphere.github.io\/marathon\/docs\/persistent-volumes.html

