## 应用容器监控

### [cAdvisor](https://github.com/google/cadvisor)

具体来说，对于每个容器，cAdvisor能够保存容器的资源隔离参数，历史资源使用，完整历史资源使用的直方图和网络统计，这些数据是从容器和宿主机导出的。

cAdvisor原生支持Docker容器，并且对任何其他类型的容器能够开箱即用。cAdvisor是基于[lmctfy](https://github.com/google/lmctfy)的容器抽象，所以容器本身是分层嵌套的。

#### 运行

通过提供的容器镜像直接运行cAdvisor非常简单。在宿主机上运行一个cAdvisor实例就可以监控宿主机和其上的所有容器。运行命令如下：

```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

上述命令让cAdvisor启动并在后台运行，该启动设置包含了cAdvisor需要监控的Docker状态的目录。可以通过`http://<宿主机IP>:8080`来访问cAdvisor提供的WEB界面。

在CentOS上运行cAdvisor时，需要添加`--privileged=true` 和 `--volume=/cgroup:/cgroup:ro`两项配置。

- CentOS/RHEL对其上的容器做了额外的安全限定，cAdvisor需要通过Docker的socket访问Docker守护进程，这需要`--privileged=true`参数。

- 某些版本的CentOS/RHEL将cgroup层次结构挂载到了`/cgroup`下，cAdvisor需要通过`--volume=/cgroup:/cgroup:ro`获取容器信息。

如果碰到`Invalid Bindmount /`错误，可能是由于Docker版本较低所致，可以在启动cAdvisor时不挂载`--volume=/:/rootfs:ro`。
