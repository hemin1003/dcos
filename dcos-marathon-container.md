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

说明：Marathon的命令行参数都对应一个添加前缀“MARATHON\_”的变量。

#### 容器应用定义示例

为了让容器镜像采用Docker容器化运行，需要在应用定义JSON中添加`container`节点配置。如下示例：

```json
{ 
    "container": { 
        "type": "DOCKER", 
        "docker": { 
            "network": "HOST", 
            "image": "group/image" 
        }, 
        "volumes": [ 
            { "containerPath": "/etc/a", "hostPath": "/var/data/a", "mode": "RO" }, 
            { "containerPath": "/etc/b", "hostPath": "/var/data/b", "mode": "RW" } 
        ] 
    }
}
```

此处`volumes`和`type`为可选配置，`type`的默认值为`DOCKER`。承载容器镜像运行的Mesos沙箱的系统挂载点可以通过环境变量$MESOS\_SANDBOX获取。

#### 桥接网络模式

桥接网络使得与容器内静态配置的端口进行绑定的应用的运行变得容易。Marathon负责Mesos管理的端口资源和Docker绑定的主机端口间的对接。

**动态端口绑定**

```json
{ 
    "id": "bridged-webapp", 
    "cmd": "python3 -m http.server 8080", 
    "cpus": 0.5, "mem": 64.0, "instances": 2, 
    "container": { 
        "type": "DOCKER", 
        "docker": { "image": "python:3", "network": "BRIDGE", 
            "portMappings": [ 
                { "containerPort": 8080, "hostPort": 0, "servicePort": 9000, "protocol": "tcp" }, 
                { "containerPort": 161, "hostPort": 0, "protocol": "udp"} 
            ] 
        } 
    }, 
    "healthChecks": [ 
        { "protocol": "HTTP", "portIndex": 0, "path": "/", "gracePeriodSeconds": 5, "intervalSeconds": 20, "maxConsecutiveFailures": 3 } 
    ] 
}
```

使用次有容器镜像

静态端口绑定





