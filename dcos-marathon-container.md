## 容器运行管理

Marathon通过Docker引擎和通用容器运行时支持Docker和AppC容器镜像的运行。

Marathon支持两种运行Docker容器镜像的方式：

* Docker容器化，使用Docker引擎运行容器镜像。

* Mesos容器化，使用通用容器运行时运行容器镜像。

### Docker容器化

Docker容器化依赖外部Docker引擎来运行Docker容器镜像。采用Docker引擎运行Docker容器镜像时，需要对Agent节点及Marathon的配置做一些调整。

#### 运行参数配置

注意，DCOS在部署时已针对Docker容器化做了参数调整，因此，下述配置仅适用于独立运行Marathon服务场景。下述参数在DCOS中同样可以找到，只是位置稍有差别。

1.调整Agent节点配置，提高Docker容器化的优先级。注意，此处配置的值顺序决定了容器运行时的选择，靠前的优先。

`$ echo 'docker,mesos' > /etc/mesos-slave/containerizers`

2.增加执行程序超时设置，以应对因网络带宽等因素将docker镜像拉取到Agent节点的潜在延迟。

`$ echo '10mins' > /etc/mesos-slave/executor_registration_timeout`

上述两项配置在DCOS中对应的位置为\(Agent节点\)：`/opt/mesosphere/etc/mesos-slave-common`。

3.调整Marathon的启动参数`--task_launch_timeout`（在杀死一个任务之前等待其进入**TASK\_RUNNING**的时间，毫秒，默认值为30000），使其匹配Agent节点执行程序超时设置。在DCOS中，该启动参数通过变量`MARATHON_TASK_LAUNCH_TIMEOUT`进行设置，参数位于\(Master节点\)：`/opt/mesosphere/etc/marathon`。



