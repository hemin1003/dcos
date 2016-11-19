## Mesos Containerizer

MesosContainerizer使用特定于Linux的功能（如控制cgroup和命名空间）提供轻量级容器化和executors的资源隔离。它是可组合的，因此开发人员可以选择性地启用不同的隔离器。它还为POSIX系统（例如OSX）提供基本支持，但没有任何实际隔离，仅提供资源使用报告。

### 共享文件系统

可以选择在Linux主机上使用**SharedFilesystem**隔离器，以允许修改每个容器针对共享文件系统的视图。这些修改信息通过框架或使用Agent参数`--default_container_info`，在`ExecutorInfo`中的`ContainerInfo`里指定。

`ContainerInfo`中指定的卷以读写（`RW`）或只读（`RO`）方式将共享文件系统（`host_path`）的部分映射到容器的文件系统（`container_path`）中。`host_path`可以是绝对路径，在这种情况下，它将使以`host_path`为根目录的文件系统子树也可以在每个容器的`container_path`下访问。如果host\_path是相对路径，则它被认为是相对于executor的工作目录。目录将被创建，并从共享文件系统中的相应目录（必须存在）中复制权限。

**SharedFilesystem**隔离器的主要目的是使共享文件系统能够选择性地的为每个容器分配部分私有资源。例如，可以使用`host_path ="tmp"`和`container_path ="/ tmp"`来实现私有的“`/tmp`”目录，其将在执行器的工作目录（模式1777）内创建目录“tmp”，并同时将其在容器内安装为`/tmp`。

### **Pid Namespace**



### **Posix Disk Isolator**

### **XFS Disk Isolator**

### **Docker Runtime Isolator**

### **The **`cgroups/net_cls`** Isolator**

### **The **`docker/volume`** Isolator**

### **The **`network/cni`** Isolator**

### **The **`linux/capabilities`** Isolator**

