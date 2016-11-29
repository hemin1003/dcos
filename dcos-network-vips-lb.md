## 基于VIPs的负载调度

在应用的端口定义中添加VIP\_$IDX标签，可以让应用服务启用4层负载（前提之一是应用通过健康检查）。应用启动时，DC\/OS会将该VIP配置分发到集群中的所有节点，这些节点都在负载平衡过程中充当决策者。集群所有节点上都运行着一个进程，当标记为该VIP的数据包到达时，此进程会跟踪这些应用服务的可用性和可访问性，以尝试向正确的后端发送请求。

### 建议

#### 警告

* 不要为节点之间的流量添加防火墙过滤。

* 不要更改ip\_local\_port\_range。

* 必须安装ipset包。

* 必须使用支持的操作系统。


#### 持久化连接

建议在使用VIPs方案时，对长连接做持久化，否则可能很快就会占满TCP套接字表。Linux上的默认允许源连接使用本地端口的范围是从32768到61000，也就是在给定源IP和目标地址端口对之间可以建立28232个连接。TCP连接必须在回收之前经过等待时间状态，Linux内核的默认TCP时间等待周期为120秒。考虑到这一点，只要以235个新连接\/秒的速度创建，则120秒左右就会耗尽连接表。

#### 健康检查

应用必须正常运行并通过健康检查时，VIPs负载调度才可用。不推荐使用TCP检查，因为它仅检查端口是否可用，可能并不能真正确认服务是正常可用的。

注意：在Docker容器应用中使用command健康检查时，command命令是在容器内运行的，如果使用类似`curl`的命令，则命令必须已安装或以`RW`模式挂载了节点上的`/opt/mesosphere`路径。

### 潜在的问题

#### IP Overlay

如果指定的VIP地址在网络中的其他位置使用，则可能会出现问题。虽然VIP是一个3元组，但最好确保专用于VIP的IP仅供负载均衡使用，而不会在网络中配置它用。因此，应选择遵循RFC1918范围内的IP。

#### IPSet

在系统内必须安装ipset。如果没有，可能会看到以下错误：

    [error] Unknown response: {ok,"iptables v1.4.21: Set minuteman doesn't exist.\n\nTry `iptables -h' or 'iptables --help' for more information.\n"}

#### 端口

端口61420和61421必须打开，负载均衡才能正常工作。由于负载均衡维护部分网格，它需要确保节点之间的连接不受阻碍。

#### 连接表耗尽

如果出现前面描述的行为正在耗尽连接表，则会在日志中看到各种错误。可以通过设置两个sysctl来缓解这个问题，但它不会消除警告。

* net.netfilter.nf\_conntrack\_tcp\_timeout\_time\_wait = 0 - 可以将其设置为0，但time\_wait状态可能会中断其他应用程序对连接的跟踪。

* net.ipv4.tcp\_tw\_reuse = 1 - 此sysctl可能是危险的，它会打破防火墙以及NAT。尽管，如果防火墙正确实现了对TCP时间戳的跟踪，这个问题将不会存在，但是不要设置net.ipv4.tcp\_tw\_recycle这个sysctl，因为它不兼容RFC，将打破防火墙对连接的跟踪。


更多信息可参考：[https:\/\/www.kernel.org\/doc\/Documentation\/networking\/ip-sysctl.txt](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)。

### 调试

负载均衡在DC\/OS集群中的每个节点上提供几个接口用于收集统计信息。接口URI为：http:\/\/localhost:61421\/metrics。

### 实现

本地进程大约每5秒轮询一次主节点，主节点也将此节点缓存5秒，因此，更新的传播时间限制为大约11秒。这种情况仅适用于新的VIPs，对于故障节点不适用。

#### 数据层面

iptables -n -L -v

#### 负载算法

#### 故障检测

