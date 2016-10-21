DCOS之系统组件服务

DCOS Master节点系统组件服务

![](/assets/dcos_system_components.png)

DCOS Agent节点系统组件服务

![](/assets/dcos_system_components_agent.png)

如上述两图所示，通过DCOS集群节点“`/etc/systemd/system/dcos.target.wants/`”目录下可以看到所有系统组件服务，Master节点和Agent节点上的服务稍有不同。详细服务说明请参考下表：

| 系统组件 | 服务名称 | 描述 |
| --- | --- | --- |
| Admin Router Agent | dcos-adminrouter-agent.service | 高性能的WEB服务器和WEB反向代理服务器，用于保存集群中所有Agent节点的列表 |
| Admin Router Master | dcos-adminrouter.service | 高性能的WEB服务器和WEB反向代理服务器，用于保存集群中所有Master节点的列表 |
| Admin Router Reloader Timer | dcos-adminrouter-\(agent\)-reload.timer | 周期性（默认每小时一次）的重启Admin Router Nginx服务，启用新的DNS解析 |
| Admin Router Service | dcos-adminrouter.service | 由Mesosphere创建的一个Nginx配置，用于中心授权，代理集群节点内部服务。Admin Router服务是DC\/OS内部核心的负载均衡服务。Admin Router是一个定制的Nginx，它在端口80上代理所有内部的服务 |
| Diagnostics | dcos-3dt.service |  |
| Diagnostics Socket | dcos-3dt.socket |  |
| DNS Dispatcher |  |  |
| DNS Dispatcher Watchdog |  |  |
| DNS Dispatcher Watchdog Timer |  |  |
| Erlang Port Mapping Daemon |  |  |
| Exhibitor |  |  |
| Generate resolv.conf |  |  |
| Generate resolv.conf Timer |  |  |
| History Service |  |  |
| Job | dcos-metronome.service | 该服务支撑DCOS的Job任务特性 |
| Layer 4 Load Balancer | dcos-minuteman.service | 该服务也称为“Minuteman”，是一个4层负载均衡实现，可以用来编排多层微服务架构 |
| Logrotate Mesos Master | dcos-logrotate-master.service | 自动循环压缩、删除和邮寄Master节点上的日志，确保不会在磁盘上存储过多日志文件 |
| Logrotate Mesos Slave | dcos-logrotate-agent.service | 自动循环压缩，删除和邮寄agent节点上的日志 |
| Logrotate Timer | dcos-logrotate-master（agent）.timer | 设置logrotate的服务间隔为2分钟 |
| Marathon | dcos-marathon.service | DCOS上的Marathon服务实例，负责启动和监控DCOS上的应用及服务 |
| Mesos Agent | dcos-mesos-slave.service | 运行在Private Agent节点上的mesos-slave进程 |
| Mesos Agent Public | dcos-mesos-slave-public.service | 该服务是运行在Public Agent节点上的mesos-slave服务进程 |
| Mesos DNS | dcos-mesos-dns.service | 该服务负责在集群内部提供服务发现功能，这是DCOS集群的内部服务，它为集群服务提供了域名“$sevice.mesos”，如一个Master Leader节点，其域名为“leader.mesos”，在集群内部就可以通过“ssh leader.mesos”登录该节点 |
| History Service | dcos-history-service.service | 该服务允许DCOS Web接口可以展现集群资源使用的统计信息 |
| Mesos Master | dcos-mesos-master.service | Mesos Master节点上的进程，负责编排Agent节点的任务 |
| Mesos Persistent Volume Discovery |  |  |
| Virtual Network Service | dcos-navstar.service | 该服务是一个守护进程，用来 |
| OAuth | dcos-oauth.service | 该服务负责DC\/OS的安全检查 |
| Package service | dcos-cosmos.service | 内部打包API服务。当每次通过CLI执行“dcos package install”时都会调用该服务。该服务将DC\/OS服务包从DC\/OS Universe部署到你的DC\/OS集群 |
| Signal | dcos-signal.service | 该组件为帮助完善DC\/OS，会周期性的向Mesosphere发送当前集群的概要信息反馈，并为集群问题提供高级检测。Signal查询Master节点上的诊断服务“\/system\/health\/v1\/report”，并将数据发送到SegmentIO，用于跟踪度量和客户支持 |
| Signal Timer | dcos-signal.timer | 设置Signal组件每小时触发一次 |
| REX-Ray | dcos-rexray.service | REX-Ray存储方案实现，让Marathon能够使用外部存储 |
| System Package Manager API | dcos-pkgpanda-api.service | 创建链接，安装systemd服务单元，为每个主机建立指定角色（Master，Private Agent， Public Agent） |
| System Package Manager API socket | dcos-pkgpanda-api.socket | System Package Manager API socket |

Diagnostics

