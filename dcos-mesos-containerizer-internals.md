## 容器化实现细节

容器化是Mesos的一个组件，负责启动容器。容器化控制着为tasks\/executors启动的容器实例，并负责它们的隔离，资源管理和事件（例如统计）。

### 容器化器的创建与启动

Mesos agent根据--containerizers参数创建容器化器。如果指定了多个容器化器，Mesos agent会创建一个组合的容器化器。

如果在TaskInfo中没有指定executor，Mesos agent将使用任务的默认executor（取决于代理使用的Containerizer，它可能是mesos-executor或mesos-docker-executor）。在Mesos的JIRA列表上有一条[MESOS-1718](https://issues.apache.org/jira/browse/MESOS-1718) BUG记录，该记录指出Command executor会导致过量使用Slave的资源，该BUG修复后，Mesos master将负责生成executor的信息。

#### **Composing Containerizer**

Composing containerizer将组合指定的containerizer（使用Agent参数--containerizers），并像单个containerizer一样工作。这是一个`compose`设计模式的实现。

#### **Docker Containerizer**

Docker容器化器使用docker中提供的docker引擎来管理容器。

**容器启动**

只有当ContainerInfo :: type设置为DOCKER时，Docker containerizer才会尝试在docker中启动该任务。

Docker containerizer将首先拉图像。

调用预启动钩子程序。

Executor将以两种方式之一启动：

1）Mesos agent在docker容器中运行

> 这通过Agent参数--docker\_mesos\_image是否存在来指示。在这种情况下，参数 --docker\_mesos\_image的值被假定为用于启动Mesos agent的docker镜像。
> 
> 如果任务包含一个executor（或自定义的executor），这个executor会在容器内启动。
> 
> 如果任务不包含一个executor，如定义了一个command，默认的mesos-docker-executor会在docker容器中启动，并通过Docker CLI执行这个command。

2）Mesos agent不在docker容器中运行

> 如果任务包含一个executor（或自定义的executor），这个executor会在容器内启动。
> 
> 如果任务不包含一个executor，如定义了一个command，任务会分出一个子进程来执行默认的mesos-docker-executor，mesos-docker-executor然后生成一个shell并通过Docker CLI来执行这个command。

#### **Mesos Containerizer**

Mesos containerizer是原生的Mesos容器化器。 Mesos Containerizer将处理任何未指定ContainerInfo :: DockerInfo的executor\/task。

**容器启动**

调用每个isolator的prepare方法。

使用启动器（Launcher）复制executor，复制的子程序在完全被隔离之前处于阻塞状态。

隔离executor。使用每个isolator的pid调用isolate方法。

获取executor。

执行executor。发出信号给复制的子程序让其继续执行。它首先执行一些初始化指令，然后执行executor。

**启动器（Launcher）**

启动器负责复制\/销毁容器。

在容器化上下文中创建一个新的进程。子进程将使用给定的argv，flags和环境在给定的路径上执行程序。

子进程的I \/ O将根据指定的I \/ O描述符进行重定向。

**Linux启动器**

* 为容器创建一个“freezer” cgroup。

* 在主机（父进程）和容器进程之间创建一个posix “pipe” 用于通信。

* 使用clone系统调用创建一个子进程（容器进程）。

* 将新创建的容器进程转移到freezer层。

* 在父进程中向管道的写入端写入一个字符来通知子进程继续（执行）。


**Posix启动器**

**Isolators**

隔离器负责为容器创建环境，其中像cpu，网络，存储和内存等资源可以与其他容器隔离。

### 容器化器的状态

#### Docker

* FETCHING
* PULLING
* RUNNING
* DESTROYING

#### Mesos

* PREPARING
* ISOLATING
* FETCHING
* RUNNING
* DESTROYING







