## 有状态应用服务

### 有状态应用的伸缩

### 有状态应用的升级或重启

### 内部机理

### 潜在的陷阱

当在Marathon中使用有状态应用时，使用动态预留和持久化卷有以下问题和限制。

#### 资源需求

目前，有状态应用的资源需求不能改变。发布APP的JSON定义后，应用的初始卷大小，cpu使用情况，内存需求等不能更改。

#### 复制和备份

因持久化卷被固定到Agent节点，如果节点与集群断开连接（如网络分区故障或Agent节点崩溃），则它们将不可访问。如果有状态服务不单独处理数据复制，则需要手动设置复制或备份策略，以防止因网络分区故障或Agent节点崩溃导致程序中的数据丢失。

如果Agent节点重新注册到集群并提供其资源，Marathon能够确保在该Agent节点上重新启动一个任务。如果节点最终不能重新注册到集群，Marathon将一直等待该节点的资源被重新提供，因为其目标是重新使用现有的数据。如果Agent节点不能按预期恢复，可以通过添加`wipe=true`参数手动删除相关任务，Marathon最终将在另一Agent节点上启动具有新持久化卷的新任务。

#### 磁盘消耗

Mesos自0.28版本开始，销毁持久性卷将不会清除或破坏数据。 Mesos仅删除有关该卷的元数据，但数据将保留在磁盘上。要防止磁盘消耗，应在不再需要时手动删除数据。

##### 非唯一的角色

Mesos中的静态和动态资源预留都是绑定到角色，而不是框架或框架实例。Marathon将添加标签以声明资源已被保留用于如上所述的frameworkId和taskId的组合。但是，这些标签不能防止其他框架或旧的Marathon实例（1.0版本之前）的滥用。每个为给定角色注册的Mesos框架将最终将接收已为该角色保留的资源供应。

然而，如果另一个框架不尊重标签和语义的存在，并使用它们，Marathon无法为最初的目的回收这些资源。如果其中一个使用动态预留，建议不要对不同的框架使用相同的角色。HA模式中的Marathon实例不需要具有唯一的角色，因为它们在设计之初就使用相同的角色。

#### Mesos沙箱

Mesos临时沙箱仍然是stdout和stderr日志的输出位置。要查看这些日志，可以从DC\/OS Web界面的Marathon管理页面获取。

### 检查和删除“挂起”状态的任务

要销毁和清除持久化卷并释放与任务相关联的预留资源，执行2个步骤：

* 定位包含持久化卷的Agent节点并删除其中的数据。

* 向Marathon发送包含`wipe=true`参数的HTTP DELETE请求。

如下示例：

```
$ http GET http://dcos/service/marathon/v2/apps/postgres/tasks 
响应: 

{ 
    "appId": "/postgres", 
    "host": "10.0.0.168", 
    "id": "postgres.53ab8733-fd96-11e5-8e70-76a1c19f8c3d", 
    "localVolumes": [ 
        { 
            "containerPath": "pgdata", 
            "persistenceId": "postgres#pgdata#53ab8732-fd96-11e5-8e70-76a1c19f8c3d" 
        } 
    ], 
    "slaveId": "d935ca7e-e29d-4503-94e7-25fe9f16847c-S1" 
}
```

根据上述信息：
通过ssh登录到Agent节点中并运行`rm -rf <volume-path>/*`命令来删除磁盘上的数据。
使用`wipe=true`删除任务，这将从Marathon内部存储库中清除任务信息，并最终销毁卷并取消保留先前与任务相关联的资源：

```
http DELETE http://dcos/service/marathon/v2/apps/postgres/tasks/postgres.53ab8733-fd96-11e5-8e70-76a1c19f8c3d?wipe=true
```

