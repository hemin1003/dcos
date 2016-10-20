Mesos-DNS

Mesos-DNS对每个mesos任务生成一个SRV记录（包括marathon应用实例）并且把这些记录翻译成机器上运行应用的IP和端口号。

当应用通过多个框架加载，而不仅仅是marathon，Mesos-DNS是非常有用的。每个容器都有个IP，类似[Project Calico](https://www.projectcalico.org/)的解决方案。在marathon中你可以随机分配端口号。



