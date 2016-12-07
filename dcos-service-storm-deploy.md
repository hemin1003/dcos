## 部署Storm集群

在DC/OS中部署Storm集群环境，主要是指部署Storm的 **Nimbus**、**UI**和**DRPC**服务。Storm的Supervisor/Worker服务是在向Storm集群部署拓扑（Topology）时，由Mesos自动分配资源调度执行。

### Nimbus部署

在本实现的镜像中，通过Marathon部署Nimbus时，Storm的UI服务可以在启动Nimbus时默认同时启动。

根据前述Storm集群配置所述，即使在`storm.yaml`中配置了不同于`mesos.master.url`的`storm.zookeeper.servers`配置，服务启动时默认仍使用`mesos.master.url`中配置的ZK集群信息，因此，在DC/OS中部署Storm集群时，根据是否使用DC/OS默认ZK集群，Marathon的应用配置稍有差别。

#### 使用DC/OS的Exhibitor(ZK)服务

在使用DC/OS默认的Exhibitor(ZK)集群服务时，`mesos.master.url`通常配置为：`zk://mesos.master:2181/mesos`。`storm.zookeeper.servers`配置为：`mesos.master`。即：

```
mesos.master.url: zk://mesos.master:2181/mesos
storm.zookeeper.servers:
 - "mesos.master"
```

Storm集群的配置信息默认存储在ZK的`/storm`节点下。

Storm集群默认配置数据目录存放在“`storm-local`”下，相对于`/opt/storm`路径。可以通过容器卷挂载将数据存放到持久化卷或外部存储，如NFS。

可以通过容器卷挂载，将`/opt/storm/conf`和`/opt/storm/log4j2`挂载到外部存储，让Storm集群能够读取外部配置。

当前在DC/OS中，Nimbus服务部署的网络采用**HOST**模式。

**注意**，当前实现无法读取`nimbus.seeds`中的配置，只支持`nimbus.host`。因此，必须在`storm.yaml`（可以通过挂载）中指定Nimbus服务的节点IP地址或主机名。这与DC/OS中服务的动态性相冲突，目前的解决方案是通过Marathon部署时[限定](/dcos-marathon-constraints.md)Nimbus的部署节点，并预先在`storm.yaml`中指定所部署的目标Agent节点的IP或主机名。

**注意**，脚本run-with-marathon.sh默认同时启动UI服务，可以通过**STORM_UI_OPTS**环境变量进行特定设置。

此种方式的Marathon应用JSON定义如下：

```json
{
  "id": "/storm-nimbus",
  "cmd": "./bin/run-with-marathon.sh",
  "env": {
    "HOSTNAME": "192.168.1.80",
  },
  "instances": 1,
  "cpus": 1,
  "mem": 1024,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.1.80"
    ]
  ],
  "container": {
    "docker": {
      "image": "192.168.0.1/daas/storm:0.2.1-SNAPSHOT-1.0.2-1.1.0-jdk8",
      "forcePullImage": true,
      "privileged": false,
      "network": "HOST"
    },
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/opt/storm/conf",
        "hostPath": "/data/storm/conf",
        "mode": "RW"
      },
      {
        "containerPath": "/opt/storm/log4j2",
        "hostPath": "/data/storm/log4j2",
        "mode": "RW"
      },
      {
        "containerPath": "/opt/storm/storm-local",
        "hostPath": "/data/storm/data",
        "mode": "RW"
      }
    ]
  },
  "healthChecks": [
    {
      "protocol": "HTTP",
      "path": "/",
      "gracePeriodSeconds": 120,
      "intervalSeconds": 20,
      "timeoutSeconds": 20,
      "maxConsecutiveFailures": 3,
      "ignoreHttp1xx": false
    }
  ],
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 6626
    },
    {
      "protocol": "tcp",
      "port": 6627
    }
  ],
  "requirePorts": true
}
```

#### 使用独立的Exhibitor(ZK)服务

使用不同于DC/OS默认Exhibitor(ZK)集群时，可以通过**STORM_NIMBUS_OPTS**环境变量直接指定ZK集群的IP或主机名列表。其它需要注意的地方与上述配置相同。

此种方式的Marathon应用JSON定义如下：

```json
{
  "id": "/storm-nimbus-dev",
  "cmd": "./bin/run-with-marathon.sh",
  "env": {
    "HOSTNAME": "192.168.1.89",
    "STORM_NIMBUS_OPTS": "-c storm.zookeeper.servers=[\"192.168.1.70\",\"192.168.1.73\",\"192.168.1.88\"]"
  },
  "instances": 1,
  "cpus": 1,
  "mem": 1024,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.1.89"
    ]
  ],
  "container": {
    "docker": {
      "image": "192.168.0.1/daas/storm:0.2.1-SNAPSHOT-1.0.2-1.1.0-jdk8",
      "forcePullImage": true,
      "privileged": false,
      "network": "HOST"
    },
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/opt/storm/conf",
        "hostPath": "/data/storm-dev/conf",
        "mode": "RW"
      },
      {
        "containerPath": "/opt/storm/log4j2",
        "hostPath": "/data/storm-dev/log4j2",
        "mode": "RW"
      },
      {
        "containerPath": "/opt/storm/storm-local",
        "hostPath": "/data/storm-dev/data",
        "mode": "RW"
      }
    ]
  },
  "healthChecks": [
    {
      "protocol": "HTTP",
      "path": "/",
      "gracePeriodSeconds": 120,
      "intervalSeconds": 20,
      "timeoutSeconds": 20,
      "maxConsecutiveFailures": 3,
      "ignoreHttp1xx": false
    }
  ],
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 16626
    },
    {
      "protocol": "tcp",
      "port": 16627
    }
  ],
  "requirePorts": true
}
```

将上述JSON定义保存为`storm-dcos-nimbus.json`，通过CLI部署服务：

```
dcos marathon app add storm-dcos-nimbus.json
```
也可以将JSON定义粘贴到WEB UI部署服务的弹出框里，进行必要的修改后直接部署。

### DRPC部署

DRPC服务在DC/OS中部署时，网络采用HOST模式。

DRPC服务节点在`storm.yaml`中是通过`drpc.servers`静态配置的，这也与服务在DC/OS中的动态特性相冲突。

解决方案类似于Nimbus服务，根据需要的DRPC服务器节点数，预先选定一部分Agent节点IP地址，这些地址是一个Agent节点IP列表的子集。在部署DRPC服务时，通过约束将服务限定在隶属于这个子集的Agent节点上。

例如，假定当前DC/OS集群中存在`1.80，1.81，1.82，1.83`四个私有Agent，在`storm.yaml`中按如下配置：

```
drpc.servers:
 - "192.168.1.80"
 - "192.168.1.81"
 - "192.168.1.82"
 - "192.168.1.83"
```

在DRPC服务的Marathon定义中使用约束`hostname:LIKE:192.168.1.8[0,1,2,3]`。

DRPC服务的Marathon应用JSON定义如下：

```json
{
  "id": "/storm-drpc-dev",
  "cmd": "bin/storm drpc -c storm.log.dir=$MESOS_SANDBOX/logs",
  "instances": 4,
  "cpus": 0.5,
  "mem": 2048,
  "constraints": [
    [
      "hostname",
      "LIKE",
      "192.168.1.8[0-9]"
    ]
  ],
  "container": {
    "docker": {
      "image": "192.168.0.1/daas/storm:0.2.1-SNAPSHOT-1.0.2-1.1.0-jdk8",
      "forcePullImage": true,
      "privileged": false,
      "network": "HOST"
    },
    "type": "DOCKER",
    "volumes": [
      {
        "containerPath": "/opt/storm/conf",
        "hostPath": "/data/storm-dev/conf",
        "mode": "RO"
      }
    ]
  },
  "healthChecks": [
    {
      "protocol": "TCP",
      "gracePeriodSeconds": 30,
      "intervalSeconds": 30,
      "timeoutSeconds": 60,
      "maxConsecutiveFailures": 3
    }
  ],
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 3772,
      "labels": {
        "VIP_0": "/storm-drpc-dev:3772"
      }
    },
    {
      "protocol": "tcp",
      "port": 3773
    },
    {
      "protocol": "tcp",
      "port": 3774
    }
  ],
  "requirePorts": true
}
```

**注意**，DRPC的应用定义中，为端口**3772**配置了VIPs，集群中部署的其他服务通过`storm-drpc-dev.marathon.l4lb.thisdcos.directory:3772`调用DRPC服务时，DC/OS自动为DRPC实现了负载均衡。

DRPC服务的部署命令参考Nimbus服务的部署。