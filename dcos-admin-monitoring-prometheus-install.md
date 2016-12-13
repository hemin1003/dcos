## 安装与配置

### 镜像安装

Prometheus的所有服务包括Server本身都有Docker镜像，可以直接从[DockerHub](https://hub.docker.com/u/prom/)仓库下载。

Prometheus镜像内部使用**9090**端口。

Prometheus默认读取"`/etc/prometheus/prometheus.yml`"配置文件，可通过挂载外部存储映射"`/etc/prometheus`"来保存配置文件。

Prometheus使用卷"`/prometheus`"存储指标数据，在实际部署时可以通过挂载外部存储来保存数据。

可通过下述示例命令启动Prometheus服务：

```
docker run -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

Prometheus的[镜像定义文件](https://hub.docker.com/r/prom/prometheus/~/dockerfile/)如下，可以根据需要对其进行自定义配置：

```
FROM        quay.io/prometheus/busybox:latest
MAINTAINER  The Prometheus Authors <prometheus-developers@googlegroups.com>

COPY prometheus                             /bin/prometheus
COPY promtool                               /bin/promtool
COPY documentation/examples/prometheus.yml  /etc/prometheus/prometheus.yml
COPY console_libraries/                     /etc/prometheus/
COPY consoles/                              /etc/prometheus/

EXPOSE     9090
VOLUME     [ "/prometheus" ]
WORKDIR    /prometheus
ENTRYPOINT [ "/bin/prometheus" ]
CMD        [ "-config.file=/etc/prometheus/prometheus.yml", \
             "-storage.local.path=/prometheus", \
             "-web.console.libraries=/etc/prometheus/console_libraries", \
             "-web.console.templates=/etc/prometheus/consoles" ]
```

### 配置


### 存储

### 扩展

