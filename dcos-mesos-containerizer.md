## 容器化

### 动机

容器化程序用于在“容器”中运行任务，而容器又用于：

* 将任务与其他正在运行的任务隔离。

* 将任务“包含”在有限资源运行时环境中运行。

* 通过编程控制任务使用的各个资源（例如，CPU，内存）。

* 在预先打包的文件系统映像中运行软件，允许它在不同的环境中运行。


### 容器化类型

Mesos对现有容器技术（例如，docker）有很好的支持，并且还提供了自己的容器技术。它还支持将不同的容器技术（例如，docker和mesos）集成在一起。

Mesos实现了以下containerizer：

* Composing

* Docker

* Mesos

* External（deprecated）


可以在Agent上通过标志--containerizers指定要使用的容器化器的类型

### **Composing containerizer**

此功能允许多个容器技术一起工作。可以在Agent上通过多个逗号分隔的容器化器名称配置`--containerizers`参数（例如，`--containerizers = mesos，docker`）来启用它。逗号分隔列表的顺序很重要，因为该容器配置列表中的第一个容器化器将用来默认启动任务。

### **Docker containerizer**

Docker 容器化允许任务在docker容器中运行。通过配置Agent节点上的参数为`--containerizers = docker`，让任务以Docker容器化方式启动。

### **Mesos containerizer**

此容器化允许任务使用由Mesos提供的可插拔的隔离器组运行，这是原生的Mesos容器化器解决方案。可以将Agent节点上的参数配置为`--containerizers = mesos`来启用它。

相比于Docker容器化方案，使用Mesos容器化有如下优势：

* 使用Mesos环境的同时不必依赖外部容器技术。

* 提供细粒度的操作系统控制（例如，由linux提供的cgroups \/namespaces）。

* 提供Mesos最新的容器技术特性。

* 额外的资源控制，如其他容器技术未能提供的磁盘使用限制。

* 支持添加任务的自定义隔离。


