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

`mesos.supervisor.suicide.inactive.timeout.secs`： 在Supervisor没有任务执行时，进程自杀前的等待时间（单位：秒），默认值：120。

`mesos.master.failover.timeout.secs`：框架故障切换的超时时间（单位：秒），默认值：2473600。

`mesos.allowed.hosts`：允许运行拓扑的Agent节点的白名单列表。

`mesos.disallowed.hosts`：不允许运行拓扑的Agent节点的黑名单列表。

`mesos.framework.role`：框架使用的[角色](/dcos-mesos-roles.md)，默认为“*”。

`mesos.framework.checkpoint`：是否启用框架的[检查点](/dcos-mesos-agent-recovery.md)，默认：false。

`mesos.offer.lru.cache.size`：LRU缓存大小，默认：1000。

`mesos.offer.filter.seconds`：过滤不用的Mesos资源供给（offers）的时间（单位：秒）。这些资源供给可以在需要时由框架重新接受。默认值：120。

`mesos.offer.expiry.multiplier`：资源供给过期时间的乘数因子（配置值 x `nimbus.monitor.freq.secs`）。默认值：2.5。

`mesos.local.file.server.port`：本地文件服务器（为Supervisor/Worker提供storm.yaml的下载）绑定的端口，默认为随机端口。

`mesos.framework.name`：框架名称。默认值：“Storm!!!”。

`mesos.framework.principal`：框架用于向Mesos注册的凭据。

`mesos.framework.secret.file`：用于保存用户凭据密钥的文件地址。密钥不能以NL结束。

`mesos.prefer.reserved.resources`：优先使用预留资源（如，“*”角色）。默认值： true。

`supervisor.autostart.logviewer`：默认值为true。如果禁用logviewer，在配置时可以将`topology.mesos.executor.mem.mb`参数的值缩减128*1.2来释放多余内存。

#### 资源配置

