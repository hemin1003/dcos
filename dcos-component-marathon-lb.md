## Marathon-LB



Marathon-LB集成了HAProxy，HAProxy提供了服务代理功能，并为TCP、HTTP应用的负载均衡，它支持SSL，HTTP压缩健康检查，Lua脚本等特性。因此，Marathon-LB既可以用作服务代理，也可以用作负载均衡和服务发现工具。



Marathon-LB是一个容器应用，它订阅了Marathon的事件总线，自动获取各个APP的信息，为每一组APP生成HAProxy配置，能够根据变更实时调整更新HAProxy的配置信息。



![](/assets/dcos_marathon_lb_topology.png)



如上图所示，Marathon-LB的应用场景非常灵活，例如：



* Marathon-LB可用在边际节点上提供负载均衡和服务发现。在DCOS中可以部署于Public节点，为入口流量做路由负载。这种情况下可以将Public节点的IP通过A记录与内部或外部DNS记录绑定。



* Marathon-LB可用于内部负载均衡和服务发现，外部流量的入口路由负载可以使用硬件设备如F5，或者是云负载如AWS的Elastic Load Balancer（ELB）。



* Marathon-LB只用于内部负载均衡和服务发现。



* 可以将内部LB和外部LB同时使用Marathon-LB，不同的服务根据需求开放给不同的Marathon-LB。





### Marathon-LB的使用



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



### 为Marathon-LB启用SSL



使用Let's Encrypt自动维护SSL Certificate，并配置marathon-lb使用cert启用SSL。



https:\/\/mesosphere.com\/blog\/2016\/04\/06\/lets-encrypt-dcos\/



https:\/\/github.com\/mesosphere\/letsencrypt-dcos




