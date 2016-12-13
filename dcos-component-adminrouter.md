## Adminrouter

![](/assets/dcos-admin-router.png)

Adminrouter是由Mesosphere创建的一个开源Nginx配置，为集群中的DC/OS服务提供中心认证和代理。

Adminrouter服务运行在Master节点上；Adminrouter Agent服务运行在Agent节点上。这两个服务在启动时分别加载`nginx.master.conf` 和 `nginx.agent.conf` 两个Nginx配置文件。

在Master节点上还运行一个`dcos-adminrouter-reload.timer`服务，默认每30秒定时重新加载一次Nginx配置来获取更新；同样，在Agent节点上也运行一个`dcos-adminrouter-agent-reload.timer`服务，默认每30秒定时重新加载一次Nginx配置。


Adminrouter通过Nginx开放的路由列表如下：

![](/assets/dcos-admin-router-table.png)

### 参考

* https://github.com/dcos/adminrouter

