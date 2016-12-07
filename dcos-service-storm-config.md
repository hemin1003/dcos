## Storm/Mesos集群配置

**Nimbus**

Storm/Mesos集群的Nimbus在启动时会读取：1）Storm的默认配置，2）`/opt/storm/conf/storm.yaml`中的配置，3）通过环境变量**STORM_NIMBUS_OPTS**设定的配置，然后进行合并，形成一个合并的配置文件存放于`$storm.local.dir/generated-conf/storm.yaml`文件。通过环境变量传递的设置会覆盖前两处的配置。

**UI**

Storm/Mesos集群的UI服务配置与Nimbus相同，差别是其环境变量为**STORM_UI_OPTS**。

**DRPC**

Storm/Mesos集群的DRPC服务配置没有特定的环境变量设置，可在启动时通过命令行参数进行额外设置。

**Supervisor，Worker**

Storm/Mesos集群的Supervisor，Worker服务实例在启动时，会从Nimbus下载最新的合并配置。

### 特定配置

#### 强制配置

1. 下述两个配置参数必须定义一个，如果同时指定了两个，优先使用Docker配置。
  
  `mesos.executor.uri`：一旦根据实际环境对代码中的storm.yaml文件进行配置并重新打包分发，你需要让Mesos执行器能够找到它。通常可以将分发包放置在HDFS上，该参数需要设定放置分发包的位置。

  `mesos.container.docker.image`：如果通过容器镜像启动Storm Supervisor/Worker，该参数配置构建的容器镜像及地址，如：`192.168.0.1/storm:0.2.1-SNAPSHOT-1.0.2-1.1.0-jdk8`。

2. `mesos.master.url`： DC/OS(Mesos) Master节点的URL地址，如：`zk://mesos.master:2181/mesos`。

3. `storm.zookeeper.servers`：Storm的Nimbus服务使用的Zookeeper服务器的地址，通常使用与`mesos.master.url`相同的配置，如：`mesos.master`。**注意，**如果要配置不同于DC/OS使用的Zookeeper集群，必须通过环境变量或命令行参数进行覆盖。即使在`/opt/storm/conf/storm.yaml`中的配置不同，服务启动时仍默认加载`mesos.master.url`中的配置。

#### 可选配置

`mesos.supervisor.suicide.inactive.timeout.secs`： 在Supervisor没有任务执行时，进程自杀前的等待时间（单位：秒），默认值为：120。

```

mesos.master.failover.timeout.secs
```

: Framework failover timeout in second. Defaults to "2473600".
mesos.allowed.hosts: Allowed hosts to run topology, which takes hostname list as a white list.
mesos.disallowed.hosts: Disallowed hosts to run topology, which takes hostname list as a back list.
mesos.framework.role: Framework role to use. Defaults to "*".
mesos.framework.checkpoint: Enabled framework checkpoint or not. Defaults to false.
mesos.offer.lru.cache.size: LRU cache size. Defaults to "1000".
mesos.offer.filter.seconds: Number of seconds to filter unused Mesos offers. These offers may be revived by the framework when needed. Defaults to "120".
mesos.offer.expiry.multiplier: Offer expiry multiplier for nimbus.monitor.freq.secs. Defaults to "2.5".
mesos.local.file.server.port: Port for the local file server to bind to. Defaults to a random port.
mesos.framework.name: Framework name. Defaults to "Storm!!!".
mesos.framework.principal: Framework principal to use to register with Mesos
mesos.framework.secret.file: Location of file that contains the principal's secret. Secret cannot end with a NL.
mesos.prefer.reserved.resources: Prefer reserved resources over unreserved (i.e., "*" role). Defaults to "true".
supervisor.autostart.logviewer: Default is true. If you disable the logviewer, you may want to subtract 128*1.2 from topology.mesos.executor.mem.mb (depending on your settings).