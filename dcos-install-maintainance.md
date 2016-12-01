## 集群维护

DC/OS集群包括Master节点和Agent节点。其中Agent节点分为**Public**和**Private**两种。对于一个已部署的节点，与DC/OS相关的文件系统目录包括：

| 目录 | 说明 |
| --- | --- |
| **/opt/mesosphere** | 包含DC/OS系统的运行脚本，依赖库和集群配置。不建议修改。 |
| **/etc/systemd/system/dcos.target.wants** | 包含了用于启动组成DC/OS系统的所有Systemd服务。 |
| **/etc/systemd/system/dcos.&lt;units&gt;** | 这组文件是/etc/systemd/system/dcos.target.wants目录下的文件拷贝（链接），这些文件必须同时存在于Systemd的system目录和dcos.target.wants目录中。 |
| **/var/lib/docker** | 包含容器镜像等Docker相关数据 |
| **/var/lib/dcos** | 包含DC/OS相关的数据 |
| **/var/lib/mesos** | 包含Mesos相关的数据 |警告⚠️：当前移动`/opt/mesosphere`目录是不被支持的，可能导致DC/OS系统的异常。

