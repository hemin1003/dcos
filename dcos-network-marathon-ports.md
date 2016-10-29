## 应用服务端口配置

首先需要再次说明的是，这里的应用服务是指Docker容器或AppC容器，如非特别指明为Docker容器，则适用于两者。此外，DCOS中的容器网络包括前述提到的BRIDGE、USER和HOST三种模式。

### 端口配置相关的属性

| 属性 | 说明 |
| --- | --- |
| containerPort | 设置容器内公开的端口。仅用于BRIDGE或USER模式下的Docker容器。 |
| hostPort | 设置容器宿主机绑定的端口。在BRIDGE或USER模式下，该配置用于映射某个容器内公开的端口。在HOST模式下，该端口即为应用服务的服务端口。 |
| servicePort | 设置用于服务查找和负载均衡（如Marathon-LB）等第三方服务所绑定的端口。Marathon自身并不使用或解释该配置。该参数是可选的，默认值为0。如果应用服务在部署时设置了该端口，用户必须自己确保该端口在整个应用服务集群内是唯一的。 |
| portMapping | 取值为端口映射数组。Docker容器在BRIDGE模式下，portMapping定义了容器内服务端口与外部宿主机及服务查找端口的桥接。一个端口映射包括：一个**hostPort**，一个**containerPort**，一个**servicePort**和一个**protocol**。hostPort未设置时默认值为0（由Marathon随机设置）。 Docker容器在USER模式下，hostPort的语义略有不同，Marathon**不会**随机分配端口，这允许容器部署到用户自定义的网络上。 |
| portDefinitions | 取值为端口定义数组，一个端口定义包括：一个**protocol**，一个**port**，一个**labels**集合。仅用于HOST模式且未定义端口映射。该参数用于替代ports数组，提供更多的端口定义信息。同一个应用中，**ports**和**portDefinitions**只能配置一个。 |
| ports | 取值为数组，仅适用于HOST模式，该参数要求主机将设置的端口数组作为资源提供。仅在没有设置端口映射时才有必要配置，且同一个应用中，**ports**和**portDefinitions**只能配置一个。 |
| protocol | 取值“tcp”，“udp”或“udp,tcp”。仅在Docker容器使用BRIDGE或USER模式时作为端口映射配置的一部分。 |
| requirePorts | 该配置提醒Marathon对提供的资源是否检查指定的端口资源是否可用。这可以确保在资源分配时Agent主机上的端口是空闲的。该配置不适用于BRIDGE和USER网络模式。 |

### 随机端口设置

在端口配置中，如果取值为0，则由Marathon为应用服务随机分配端口。但是，如果在“**portMapping**”中，“containerPort”参数设置为0，Marathon会将“containerPort”和“hostPort”取相同的值。

### 环境变量

Marathon会将每个应用配置的宿主机端口值以环境变量$PORT0，$PORT1，...的形式提供给应用容器环境。每一个Marathon应用默认都会分配一个端口，所以$PORT0总是可访问的。这些变量在Marathon运行的Docker容器内也是可用的。此外，如果某个端口设置了NAME名称，则也可以通过$PORT\_NAME变量访问。

在BRIDGE或USER网络模式下，要确保应用服务绑定到了“portMapping”中定义的“containerPort”参数。如果“containerPort”参数取值为0，其值与“hostPort”相同，可用通过$PORT变量来访问。

### 端口配置示例

HOST网络模式



BRIDGE网络模式

















