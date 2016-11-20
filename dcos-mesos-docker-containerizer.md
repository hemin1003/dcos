## Docker Containerizer

Mesos自0.20.0版开始支持运行含有Docker镜像的任务，并支持配置一部分Docker选项。用户可以以Task或Executor的方式启动Docker镜像。

### 安装

要运行Agent时启用Docker Containerizer，必须使用“docker”作为containerizer选项之一启动代理。例如：

```
mesos-agent --containerizers=docker,mesos
```

每个支持Docker containerizer的Agent都应该安装Docker CLI客户端（version&gt; = 1.0.0）。

如果在Agent上启用iptables，通过添加以下规则确保iptables允许来自docker bridge接口的所有流量：

```
iptables -A INPUT -s 172.17.0.0/16 -i docker0 -p tcp -j ACCEPT
```

### 使用Docker Containerizer

Mesos自0.20.0版本在TaskInfo和ExecutorInfo中添加了ContainerInfo字段，这可以让Containerizer如Docker来运行一个Task或一个Executor。

要将Docker镜像作为任务运行，在TaskInfo中，必须同时设置命令和容器字段，因为Docker Containerizer将使用附带的命令启动docker镜像。ContainerInfo应该将类型设置为Docker并且将DockerInfo设置为所需docker镜像。

要将Docker镜像作为Executor执行，在TaskInfo中，必须设置一个ExecutorInfo，该ExecutorInfo必须包含一个类型为Docker的ContainerInfo，并同时设置一个CommandInfo用于启动Executor。注意，Docker镜像将作为一个Mesos的Executor启动，一旦启动，就会向Agent注册。

### Docker Containerizer的运行

Docker Containerizer负责将任务\/执行器`Launch`和`Destroy`调用转换为Docker CLI命令。目前Docker Containerizer在启动任务时将执行以下操作：

1. 将CommandInfo中指定的所有文件提取到沙盒中。

2. 从远程容器仓库中拉取docker镜像。

3. 使用Docker执行器运行docker镜像，将sandbox目录映射到Docker容器中，并将目录设置为MESOS\_SANDBOX环境变量。执行器还将容器日志写入到沙箱中的stdout\/stderr文件。

4. 在容器退出或containerizer销毁时，停止和删除docker容器。


Docker Containerizer启动所有带有mesos-前缀和Agent的id（即：mesos-agent1-abcdefghji）的容器，并且还假定所有具有mesos-前缀的容器都由Agent管理，可以自由停止或终止容器。

当Docker镜像作为执行器启动时，唯一的区别是它忽略启动命令执行器，只获取docker容器执行器的pid。

### 私有容器仓库

Mesos自1.0版本开始支持使用Docker的config.json文件设置访问私有镜像仓库的验证信息。这些验证信息可以通过参数--docker\_config传递给Agent。参数--docker\_config既可以接收文件路径，也可以直接设置JSON对象：

```
--docker_config=file:///home/vagrant/.docker/config.json
```

或

```
--docker_config="{ \
  \"auths\": { \
    \"https://index.docker.io/v1/\": { \
      \"auth\": \"xXxXxXxXxXx=\", \
      \"email\": \"username@example.com\" \
    } \
  } \
}"
```

除了给Agent传递参数，另一种方式是在APP的JSON定义中同uri提供docker的config.json的位置，详细信息可参考“[容器运行管理](/dcos-marathon-container.md)”。

### **CommandInfo在运行Docker镜像时的作用**

参考“[容器运行管理](/dcos-marathon-container.md)”章节。

### 在Agent恢复时恢复Docker容器

无论Agent节点本身是否在Docker容器中运行，Docker容器化器都支持在Agent节点重新启动时恢复Docker容器。当启用--docker\_mesos\_image参数时，Docker容器化器假定其本身在容器中运行，并相应地修改其恢复的机制来启动docker容器。

