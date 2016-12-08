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

#### WEB UI

cAdvisor提供了一个WEB界面可以直观的查看容器的状态信息：

![](/assets/dcos-cadvisor-webui.png)

如果需要添加安全验证，可以有两种方案：

- **HTTP基本认证**

  需要使用`http_auth_file`参数指定一个通过**`htpassword`**命令生成的身份认证秘钥的文件。默认情况下，auth域设置为`localhost`。

  ```
./cadvisor --http_auth_file test.htpasswd --http_auth_realm localhost
```
  test.htpasswd的内容示例：

  ```
admin:$apr1$WVO0Bsre$VrmWGDbcBV1fdAkvgQwdk0
```

- **HTTP摘要认证**

  需要使用`http_digest_file`参数指定一个通过**`htdigest`**命令生成的身份摘要信息的文件。默认情况下，auth域设置为`localhost`。

  ```
./cadvisor --http_digest_file test.htdigest --http_digest_realm localhost
```

  test.htdigest的内容示例：

  ```
admin:localhost:70f2631dded4ce5ad0ebbea5faa6ad6e
```

如果同时设置了这两种认证，只有HTTP基本认证被启用。


#### 配置参数

#### [镜像定义](https://github.com/google/cadvisor/blob/master/deploy/Dockerfile)

```
FROM alpine:3.4
MAINTAINER dengnan@google.com vmarmol@google.com vishnuk@google.com jimmidyson@gmail.com stclair@google.com

ENV GLIBC_VERSION "2.23-r3"

RUN apk --no-cache add ca-certificates wget device-mapper && \
    apk --no-cache add zfs --repository http://dl-3.alpinelinux.org/alpine/edge/main/ && \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub && \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-${GLIBC_VERSION}.apk && \
    wget https://github.com/andyshinn/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-bin-${GLIBC_VERSION}.apk && \
    apk add glibc-${GLIBC_VERSION}.apk glibc-bin-${GLIBC_VERSION}.apk && \
    /usr/glibc-compat/sbin/ldconfig /lib /usr/glibc-compat/lib && \
    echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' >> /etc/nsswitch.conf && \
    rm -rf /var/cache/apk/*

# Grab cadvisor from the staging directory.
ADD cadvisor /usr/bin/cadvisor

EXPOSE 8080
ENTRYPOINT ["/usr/bin/cadvisor", "-logtostderr"]
```

通过cAdvisor默认的镜像定义文件可以看出，无法在运行镜像时传递自定义的启动参数。