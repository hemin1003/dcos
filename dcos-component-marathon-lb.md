## Marathon-LB

Marathon-LB（MLB）是一个根据Marathon应用状态自动调整和配置HAProxy的工具。HAProxy是一个为TCP、HTTP应用提供快速，高效，高可用性的负载均衡器，它还具有很多其他高级特性如支持SSL，HTTP压缩，健康检查，Lua脚本等。因此，Marathon-LB既可以用作服务代理，也可以用作负载均衡和服务发现工具。

Marathon-LB是一个容器应用，它订阅了Marathon的事件总线，自动获取各个APP的信息，为每一组APP生成HAProxy配置，能够根据变更实时调整更新HAProxy的配置信息。

### 主要特性

* 无状态设计：除了通过Marathon获取应用状态信息，没有直接依赖任何第三方状态存储像ZooKeeper或etcd。

* 幂等和确定性：可水平伸缩

* 高度可扩展：可以为单个实例提供端口限速，为多个实例提供容错和更高的吞吐量

* 通过Marathon的事件总线实现实时LB更新

* 支持Marathon的[健康检查](/dcos-marathon-health-checks.md)

* 多证书TLS \/ SSL支持

* 零停机部署

* 支持为每个服务提供HAProxy模板

* 与DC\/OS集成

* 可在启动时提供全局HAProxy模板

* 支持与每容器固定IP方案进行集成，如通过[Calico](https://github.com/projectcalico/calico-containers)项目


### Marathon-LB的应用

![](/assets/dcos_marathon_lb_topology.png)

如上图所示，Marathon-LB的应用场景非常灵活，例如：

* Marathon-LB可用在边际节点上提供负载均衡和服务发现。在DCOS中可以部署于Public节点，为入口流量做路由负载。这种情况下可以将Public节点的IP通过A记录与内部或外部DNS记录绑定。

* Marathon-LB可用于内部负载均衡和服务发现，外部流量的入口路由负载可以使用硬件设备如F5，或者是云负载如AWS的Elastic Load Balancer（ELB）。

* Marathon-LB只用于内部负载均衡和服务发现。

* 可以将内部LB和外部LB同时使用Marathon-LB，不同的服务根据需求开放给不同的Marathon-LB。


#### 应用示例 {#mlb-sample-config}

**注意：**默认配置下，Marathon-LB部署在Public节点上。如果需要用作内部负载均衡，可参考如下配置：

```
{
 "marathon-lb":{
   "name":"marathon-lb-internal",
   "haproxy-group":"internal",
   "bind-http-https":false,
   "role":""
 }
}
```

Marathon-LB在部署时，默认占用80，443，9090，9091，10000-10100端口（80、443端口已被Marathon-LB中的 HAProxy独占）。

要使用Marathonn-LB，服务定义中必须设置`HAPROXY_GROUP`标签。

Marathon-LB运行时绑定在服务定义的服务端口`servicePort`（如果APP不定义servicePort，Marathon会随机分配端口号）上，然后，外部或内部服务请求可以通过Marathon-LB所在节点的相关服务端口访问具体的服务。

例如：Marathon-LB部署在node1上，服务S1部署在node2上并且绑定的servicePort是10001，服务S2部署在node3上，那么S2可以通过node1上的100001端口访问部署在node2上的S1。

S1：

```
{
 "id": "S1",
 "container": {
   "type": "DOCKER",
   "docker": {
     "image": "service:1.0.0",
     "network": "BRIDGE",
     "portMappings": [
       { "hostPort": 0, "containerPort": 8080, "servicePort": 10001 }
     ],
     "forcePullImage":true
   }
 },
 "instances": 1,
 "cpus": 0.1,
 "mem": 65,
 "labels":{ "HAPROXY_GROUP":"external" }
}
```

通过增加`instances`的值，可以增加S1的部署实例，这些实例通过Marathon-LB进行负载调度。

### 虚拟主机

Marathon-LB的另一个重要特性是支持虚拟主机。这一特性可以将HTTP请求准确路由到多个主机节点（FQDNs）中的一个。

例如：网络拓扑中存在ilovesteak.com和steaknow.com两个域名，这两个DNS域名都指向**同一个Marathon-LB的同一个端口**，Marathon-LB会根据请求域名的不同，将HTTP请求提交给所负载的具体的服务节点。

S1:

```
{
 "id": "S1",
 "labels":{
   "HAPROXY_GROUP":"external",
   "HAPROXY_0_VHOST":"ilovesteak.com"
 }
}
```

S2:

```
{
 "id": "S2",
 "labels":{
   "HAPROXY_GROUP":"external",
   "HAPROXY_0_VHOST":"steaknow.com"
}
```

通过添加`HAPROXY_{n}_VHOST`（WEB虚拟主机）标签，Marathon-LB会自动把服务以虚拟主机的形式公布出来。访问域名`steaknow.com`时，会直接访问该域名绑定的Marathon-LB的IP和端口，并通过Marathon-LB自动跳转到S2服务。标签中的“0”对应着servicePort的索引，如果有多个servicePort定义，可以相应的配置为0，1，2等等。

### 最佳实践

* 尽量使用保留范围内的服务端口（默认值为10000到10100），可以避免端口冲突，确保重新加载后不会导致连接错误。

* 避免使用`HAPROXY_ {n} _PORT`标签；尽量定义服务端口。

* 考虑运行多个MLB实例。在实践中，应该使用3个或更多个来为生产工作负载提供高可用性。不推荐运行1实例，另外，除非有重大的负载运行，否则也没必要超过5个实例。运行的MLB实例的数量将根据工作负载和所需的故障容差量而变化。注意：不要在群集中的每个节点上运行MLB。

* 考虑在MLB前面使用专用的负载平衡器以便于升级\/更改。常见的选择包括用于本地安装的ELB（在AWS上）或F5等硬件负载平衡器。

* 使用单独的MLB分组（使用`--group`指定）区分内部和外部负载平衡。在DC\/OS上，默认分组是`external`。用于内部负载均衡的配置示例参考[上文](#mlb-sample-config)。

* 对于HTTP服务，请考虑设置VHost（或可选的路径）以访问端口80和443上的服务。或者，可以使用HTTP头`X-Marathon-App-Id`在端口9091上访问服务。例如，访问ID为`tweeter`的应用：

```
$ curl -vH "X-Marathon-App-Id: /tweeter" marathon-lb.marathon.mesos:9091/ 
* Trying 10.0.4.74... 
* Connected to marathon-lb.marathon.mesos (10.0.4.74) port 9091 (#0) 
> GET / HTTP/1.1 > Host: marathon-lb.marathon.mesos:9091 
> User-Agent: curl/7.48.0 
> Accept: */* 
> X-Marathon-App-Id: /tweeter 
> 
< HTTP/1.1 200 OK
```

* Marathon-LB的一些特性假定它是在自己的PID命名空间中运行的唯一实例。例如，MLB假定它在容器中运行。如果多个MLB实例在同一个PID命名空间中，或者在其他HAProxy进程的相同的PID命名空间中运行，某些功能（如`/_mlb_signa`l接口和`/_haproxy_getpids`接口）（以及扩展和零停机部署）可能会产生不可预期的行为。

* 可以考虑为环境变量`HAPROXY_RELOAD_SIGTERM_DELAY`设置为一个值，例如5m。这个值会直接传递给每次HAProxy重新加载之后执行的sleep命令，待达到延迟时间设定后再向旧的HAProxy的PID发送SIGTERM（参见[service\/haproxy\/run](https://github.com/mesosphere/marathon-lb/blob/master/service/haproxy/run)）。特别是对于需要TCP长连接的情况，更希望在所有连接完成之后再终止HAProxy。如果过于频繁地重新加载HAProxy，因PID会在指定的延迟内被重复使用，则可能导致SIGTERM发送给错误的PID。有关HAProxy重新加载的更多信息，以及更多相关问题[＃5](https://github.com/mesosphere/marathon-lb/issues/5)，[＃71](https://github.com/mesosphere/marathon-lb/issues/71)，[＃267](https://github.com/mesosphere/marathon-lb/issues/267)，[＃276](https://github.com/mesosphere/marathon-lb/issues/276)和[＃318](https://github.com/mesosphere/marathon-lb/issues/318)的更多信息，请参阅[此讨论](http://www.serverphorums.com/read.php?10,862139)。

