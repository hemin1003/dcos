## Spartan

DNS是DC/OS的一个重要的组成部分。因为在DC/OS中，任务会经常移动（Agent节点内或Agent节点之间），一些通过名称引用的资源必须被动态解析（特别是通过某些应用协议）。没有在每个项目中实现Zookeeper或Mesos客户端，DC/OS选择DNS作为所有组件之间相互发现发现的通用语言。

**[Spartan](https://github.com/dcos/spartan)**是一个遵循RFC5625规范的DNS代理/分发器，它基于Mesos DNS（一个运行在DC/OS所有的Master节点上的服务）服务实现。

在所有的子系统（Agent节点及Agent节点上所有运行的容器）中，DC/OS将每个Master节点上的Mesos DNS服务地址添加到`/etc/resolv.conf`中。如果其中一个Master节点故障，对该Master节点的DNS查询将超时，这是有问题的。Spartan通过将DNS查询双调度到多个Master节点并返回第一个结果来解决这个问题。

除此之外，为了降低风险，Spartan将查询操作路由到它认为最适合执行查询的节点。具体来说，如果域以`mesos`结束，它只会将查询分派给Mesos的Master节点；如果没有，它将发送查询请求到配置的2个上行DNS代理（初始安装DC/OS时，在安装UI界面或`genconf/config.yaml`文件中配置的两个`resolvers`）。

### 参考

https://github.com/dcos/spartan

