## 容器网络方案

容器网络需要面对的最重要的一个问题是容器的IP地址。DCOS的容器网络在Mesos V0.25版本（2015年10月）之前没有明确的方案，需要手动维护创建。在Mesos V0.25到V1.0（2016年7月）这段时期，Docker容器技术推出了网络扩展功能，Docker容器有了原生的网络扩展支持，典型的第三方插件有Weave，Calico等，此阶段可以通过Marathon的API在创建容器的同时赋予一个IP地址。在Mesos V1.0之后，Mesos原生支持了CNI网络，无论是Docker容器还是AppC容器，一容器一IP变得非常简单。

### VIPs

DCOS提供了一个名为Minuteman的服务器端内部微服务之间的东-西向（区分于Client-Server的南-北向）4层负载均衡。为了易于服务的配置和发现，DCOS采用**命名（name-based）VIPs**来定位服务。因此，客户端访问服务时连接的是一个服务地址而不是具体的IP地址，同时DCOS可以很容易的将指向一个命名VIP的调用请求映射到多个具体的IP地址和端口，从而实现负载调度。采用命名VIPs的另一个好处是可以避免与基于IP的VIP产生冲突，在服务安装时可以自动创建。

一个命名VIP包含3个组成部分：

* 私有的虚拟IP地址

* 端口（通过该端口提供服务）

* 服务名称


VIPs的命名遵循如下规则：

`<service-name>.marathon.l4lb.thisdcos.directory:<port>`

根据Marathon所运行的是Docker容器镜像还是AppC容器镜像的不同，在Marathon应用定义中分别用`portMappings`和`portDefinitions`两个不同的属性节点进行配置。

**AppC容器配置**

```
{
    "id": "myservice",
    "portDefinitions": [ 
        { 
            "protocol": "tcp", 
            "port": 6666, 
            "labels": { "VIP_0": "myservice:6666" }, 
            "name": "jmx" 
        }, 
        { 
            "protocol": "tcp", 
            "port": 7777, 
            "labels": { "VIP_1": "myservice:7777" }, 
            "name": "api" 
        } 
    ]
}
```

如上示例，该配置定义了两个VIP：

* `myservice.marathon.l4lb.thisdcos.directory:6666`

* `myservice.marathon.l4lb.thisdcos.directory:7777`


客户端（注：此处指DCOS集群中的其他服务）可以通过调用端口为6666的VIP访问服务提供的JMX管理功能，可以通过调用端口为7777的VIP访问服务提供的业务API。

上述配置也可以通过DCOS的WEB管理控制台进行配置：

![](/assets/dcos_network_vip_appc.png)

**Docker容器配置**

```
{ 
    "id": "myservice", 
    "container": { 
        "docker": { 
            "image": "chrisrc/myservice", 
            "forcePullImage": false, 
            "privileged": false, 
            "portMappings": [ 
                { 
                    "containerPort": 6666, 
                    "protocol": "tcp", 
                    "name": "jmx", 
                    "servicePort": 6666, 
                    "labels": { "VIP_0": "myservice:6666" } 
                }, 
                { 
                    "containerPort": 7777, 
                    "protocol": "tcp", 
                    "name": "api", 
                    "servicePort": 7777, 
                    "labels": { "VIP_1": "myservice:7777" } 
                } 
            ], 
            "network": "BRIDGE" 
        } 
    }
}
```

如上示例，该配置定义了一个名为myservice的服务，该服务通过Docker镜像chrisrc\/myservice提供。与上例类似，该服务也定义了两个客户端（注：此处指DCOS集群中的其他服务）可以直接访问的VIP：

* `myservice.marathon.l4lb.thisdcos.directory:6666`

* `myservice.marathon.l4lb.thisdcos.directory:7777`


上述配置也可以通过DCOS的WEB管理控制台进行配置：

![](/assets/dcos_network_vip_docker.png)

