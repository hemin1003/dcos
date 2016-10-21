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
| Job |  |  |
| Layer 4 Load Balancer |  |  |
| Logrotate Mesos Master |  |  |
| Logrotate Mesos Slave |  |  |
| Logrotate Timer |  |  |
| Marathon |  |  |
| Mesos Agent |  |  |
| Mesos Agent Public |  |  |
| Mesos DNS |  |  |
| History Service |  |  |
| Mesos Master |  |  |
| Mesos Persistent Volume Discovery |  |  |
| Virtual Network Service |  |  |
| OAuth |  |  |
| Package service |  |  |
| Signal |  |  |
| Signal Timer |  |  |
| REX-Ray |  |  |
| System Package Manager API |  |  |
| System Package Manager API socket |  |  |





Diagnostics

