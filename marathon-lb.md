Marathon-LB

实现服务发现的另外一种方法是在集群的每个主机上运行一个TCP\/HTTP的代理，透明地连接本地主机上的静态服务端口转发到个人的动态分配主机\/端口组合马拉松应用程序实例\(运行便任务\)。客户端很容易连接到服务端口，并且不需要知道服务发现实现的具体细节。如果所有的应用通过marathon加装，那这个方式就足够了。

Marathon-lb是一个**[Docker](http://lib.csdn.net/base/4 "Docker知识库")**化的应用，包含HAProxy，使用marathon rest api重新生成HAProxy配置。它支持高级功能像SSL卸载,粘性的连接,和基于VHost负载平衡,允许为你的marathon应用指定虚拟host。

当使用marathon-lb，注意requirePosts=true不是不须设置，其他的说明在[ports documentation](https://mesosphere.github.io/marathon/docs/ports.html)中查阅。



Marathon-lb既是一个服务发现工具，也是负载均衡工具，它集成了haproxy，自动获取各个app的信息，为每一组app生成haproxy配置，通过servicePort或者web虚拟主机提供服务。

要使用marathonn-lb，每组app必须设置HAPROXY\_GROUP标签。

Marathon-lb运行时绑定在各组app定义的服务端口（servicePort，如果app不定义servicePort，marathon会随机分配端口号）上，可以通过marathon-lb所在节点的相关服务端口访问各组app。

例如：marathon-lb部署在slave5，test-app 部署在slave1，test-app 的servicePort是10004，那么可以在slave5的 10004端口访问到test-app提供的服务。

由于servicePort 非80、443端口（80、443端口已被marathon-lb中的 haproxy独占），对于web服务来说不太方便，可以使用 haproxy虚拟主机解决这个问题：

在提供web服务的app配置里增加HAPROXY\_{n}\_VHOST（WEB虚拟主机）标签，marathon-lb会自动把这组app的WEB集群服务发布在marathon-lb所在节点的80和443端口上，用户设置DNS后通过虚拟主机名来访问。



External

需求：至少1个Public Node

Internal

使用Let's Encrypt自动维护SSL Certificate，并配置marathon-lb使用cert启用SSL。

https:\/\/mesosphere.com\/blog\/2016\/04\/06\/lets-encrypt-dcos\/

https:\/\/github.com\/mesosphere\/letsencrypt-dcos

