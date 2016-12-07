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

**注意，**当前实现无法读取`nimbus.seeds`中的配置，只支持`nimbus.host`。因此，必须在`storm.yaml`（可以通过挂载）中指定Nimbus服务的节点IP地址或主机名。这与DC/OS中服务的动态性相冲突，目前的解决方案是通过Marathon部署时[限定](/dcos-marathon-constraints.md)Nimbus的部署节点，并预先在`storm.yaml`中指定所部署的目标Agent节点的IP或主机名。

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

### DRPC部署
