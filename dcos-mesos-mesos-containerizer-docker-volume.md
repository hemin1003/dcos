### Docker Volume Support

### 目的

### 工作机理

docker\/volume隔离器使用dvdcli（来自EMC的开源命令行工具）与Docker卷插件交互。

当启动使用Docker卷的新任务时，docker\/volume隔离器将调用dvdcli将相应的Docker卷挂载到主机上，然后挂载到容器上。

当任务完成或被杀死时，docker\/volume隔离器将调用dvdcli以卸载相应的Docker卷。

docker\/volume隔离器的详细工作流程如下：

* 框架在启动任务时在ContainerInfo中指定外部卷。

* Master向Agent发送启动任务的消息。

* Agent接收消息，并要求所有隔离器（包括docker\/volume隔离器）为提供了ContainerInfo的容器做准备。

* 隔离器调用dvdcli将相应的外部卷挂载到主机上的挂载点。

* Agent启动容器，并将卷绑定挂载到容器中。

* 容器完成后，容器中的绑定挂载的卷将自动从容器中卸载，因为容器在其自己的挂载命名空间中。

* Agent调用隔离器进行清除，后者随之调用dvdcli以卸载容器的所有挂载点。


### 配置

### 局限

### 示例

