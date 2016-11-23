## 存储策略与方案

为了达到应用在DCOS上动态迁移和扩展的目的，DCOS上的应用在停止并重启后默认会失去状态数据而全新启动。但是，对于某些场景，如数据库，Kafka，Cassandra，Nginx缓存等有状态的应用，这些应用要求在重启时能够恢复原来的状态，此时就要考虑为这些应用提供持久化的存储策略和方案。

从容器的角度来看存储可以分为：**容器内存储**和**宿主机存储**。而宿主机存储又区分为：**本机物理存储**和**外部共享存储**。因此，从容器访问存储资源的可达性，容器数据的存储策略涉及：容器内存储资源，宿主本机存储和宿主外部共享存储三类。

### 容器内存储资源

![](/assets/task-localstorage.png)

容器内存储资源随着容器的停止而消失，因此不过多讨论。这里重点关注容器的宿主本机存储和宿主外部共享存储两类方案。

### 宿主本机存储

![](/assets/task-localstorage-localfs.png)

### 宿主外部共享存储

宿主外部共享存储也区分为两种方案，一种是分布式文件系统，一种是SAN或SDS。

#### 分布式文件系统

![](/assets/task-localstorage-dist-nfs.png)

#### SAN\/SDS

![](/assets/task-localstorage-san-sds.png)

### 参考

https:\/\/clusterhq.com\/2016\/04\/14\/persistence-volumes-mesos\/

