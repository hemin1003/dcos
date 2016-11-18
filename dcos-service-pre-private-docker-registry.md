## 搭建私有容器仓库

在DCOS中管理和使用镜像的方式有多种，各有优劣。但是，在DCOS集群内部维护一个私有容器仓库仍然是值得推荐的。它有两个主要优势：

* 它仅限于你的DCOS集群范围内，安全可控。

* DCOS集群对容器镜像依赖较重且镜像文件都比较大，可以节省带宽，减少镜像拉取和推送的时间，加快部署速度。


### 私有容器仓库的选择

从容器仓库的实现产品来讲，当前支持容器仓库的产品包括：

* Docker Trusted Registry（Docker）

* Registry（Docker）

* Artifactory

* Gitlab

* Nexus3


从安全角度，因Docker守护程序默认强制要求仓库处于可信的环境，在具体实践时，根据环境的可信度，通常有三种方案：

* 可信证书颁发机构

* 自签名证书

* 非安全


容器仓库服务作为DCOS集群内部的一个服务，必须很容易的供内部访问，同时也可能面临来自部分集群外部的访问（如推送部分外部基础镜像到容器仓库）。在DCOS中，服务命名与发现策略有：

* Mesos-DNS

* Layer 4 Load Balancer

* Marathon-LB


本方案的选择充分结合DCOS集群的特性，采用Docker官方提供的Registry镜像，自签名证书和VIPs + Marathon-LB的方式。

### 主要搭建步骤

集群环境准备。

创建具有正确的公用名和主题备用名的自签名证书。

使用证书的公钥和私钥配置Docker Registry服务。

在所有Agent节点上配置Docker守护程序以信任自签名证书。

部署Docker Registry服务。

在集群内部使用容器仓库。

在集群外部访问容器仓库。

### 集群环境准备

为便于后续命令可以同时在所有Agent节点上执行，可根据自己熟悉的方法构建一个运维节点到所有Agent节点之间的可信任的SSH连接。一种方法可参考[SSHing into Nodes](https://dcos.io/docs/1.9/administration/access-node/sshcluster/)，也可以选择在一台Master上配置公私钥并将公钥添加到所有Agent节点的authorized\_keys列表中。

使用下述配置调整Layer 4负载均衡（放弃对TCP状态的途中检测），让所有节点都能够从Registry中拉取和推送镜像。

```
for i in $(curl -sS master.mesos:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"'); do ssh "$i" -oStrictHostKeyChecking=no "sudo sysctl -w net.netfilter.nf_conntrack_tcp_be_liberal=1"; done
```

### 使用自签名证书搭建私有容器仓库

#### 创建自签名证书

确保创建自签名证书的主机上安装了OpenSSL（本例使用DCOS集群中的一个Master节点，OpenSSL位于`/opt/mesosphere/packages/openssl--8042860cf76ca9ef965af9ee6d59acace266356e`。

创建一个`certs`目录：

```
cd ~
mkdir  certs
cd certs
cp /opt/mesosphere/packages/openssl--8042860cf76ca9ef965af9ee6d59acace266356e/etc/ssl/openssl.cnf ./openssl.cnf
sed -i "/\[ v3_ca \]/a subjectAltName = IP:192.168.0.1" ./openssl.cnf
openssl req -config ./openssl.cnf -newkey rsa:2048 -nodes -keyout domain.key -x509 -days 365 -out domain.crt -subj "/C=CN/ST=SH/L=Shang Hai/O=example.com/CN=192.168.0.1"
```

使用与Docker配合工作的VIP创建自签名证书有点复杂，因为必须要修改OpenSSL的配置以将VIP包含在证书的主题备用名称中。

**注1：**此处IP“192.168.0.1”为**虚拟**IP，仅在DCOS集群内部可见，因此可以根据需要使用任意自定义的IP地址。该配置对应的Marathon JSON定义请参考本章后续小节。

**注2：**除了直接使用VIP，还可以使用服务的Mesos-DNS名称或引用第4层负载均衡VIP的名称创建证书。

使用Mesos DNS：

```
openssl req -config ./openssl.cnf -newkey rsa:2048 -nodes -keyout domain.key -x509 -days 365 -out domain.crt -subj "/C=CN/ST=SH/L=Shang Hai/O=example.com/CN=registry.marathon.mesos"
```

使用引用VIP的主机名：

```
openssl req -config ./openssl.cnf -newkey rsa:2048 -nodes -keyout domain.key -x509 -days 365 -out domain.crt -subj "/C=CN/ST=SH/L=Shang Hai/O=example.com/CN=registry.example.com"
```

#### 拷贝证书和私钥到所有Agent节点

使用下述命令将证书和私钥拷贝到集群所有Agent节点。注意，下述命令仅适用于从DCOS集群的Master节点中的当前Leader节点进行操作。

```
MESOS_AGENTS=$(curl -sS master.mesos:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"'); 

for i in $MESOS_AGENTS; do ssh "$i" -oStrictHostKeyChecking=no "sudo mkdir --parent /etc/privateregistry/certs/"; done 

for i in $MESOS_AGENTS; do scp -o StrictHostKeyChecking=no ./domain.* "$i":~/; done 

for i in $MESOS_AGENTS; do ssh "$i" -oStrictHostKeyChecking=no "sudo mv ./domain.* /etc/privateregistry/certs/"; done
```

配置所有Agent节点上的Docker守护程序信任为私有容器仓库创建的自签名证书。

```
MESOS_AGENTS=$(curl -sS master.mesos:5050/slaves | jq '.slaves[] | .hostname' | tr -d '"');

for i in $MESOS_AGENTS; do ssh "$i" -oStrictHostKeyChecking=no "sudo mkdir --parent /etc/docker/certs.d/192.168.0.1"; done 

for i in $MESOS_AGENTS; do ssh "$i" -oStrictHostKeyChecking=no "sudo cp /etc/privateregistry/certs/domain.crt /etc/docker/certs.d/192.168.0.1/ca.crt"; done 

for i in $MESOS_AGENTS; do ssh "$i" -oStrictHostKeyChecking=no "sudo systemctl restart docker"; done
```

**注1：**上述命令重启Docker引擎服务时会导致当前正在运行的容器服务重启。
**注2：**对于Docker in Docker（DnD），可以将容器的`/etc/docker/certs.d`映射到Agent节点宿主机的相同路径。

#### 部署Registry到DCOS集群

```json
{ 
    "id": "/registry", "cpus": 0.5, "mem": 128, "disk": 0, "instances": 1, 
    "constraints": [["hostname","LIKE","192.168.1.80"]], 
    "coainer": { 
        "docker": { 
            "image": "registry", "forcePullImage": false, "privileged": false, 
            "portMappings": [ { 
                "containerPort": 5000, "protocol": "tcp", "servicePort": 5000, 
                "labels": { "VIP_0": "192.168.0.1:443" } 
            } ], 
            "network": "BRIDGE" 
        }, 
        "type": "DOCKER", 
        "volumes": [ 
            { "containerPath": "/certs/", "hostPath": "/etc/privateregistry/certs/", "mode": "RO" }, 
            { "containerPath": "/var/lib/registry", "hostPath": "/data/docker-registry", "mode": "RW" } 
        ] 
    }, 
    "portDefinitions": [ { "port": 5000, "protocol": "tcp", "labels": {} } ], 
    "healthChecks": [ 
        { "protocol": "TCP", "portIndex": 0, "gracePeriodSeconds": 60, "intervalSeconds": 60, "timeoutSeconds": 20, "maxConsecutiveFailures": 3} 
    ], 
    "labels": {"HAPROXY_GROUP": "external"}, 
    "env": { 
        "REGISTRY_HTTP_TLS_CERTIFICATE": "/certs/domain.crt", 
        "REGISTRY_HTTP_TLS_KEY": "/certs/domain.key", 
        "REGISTRY_HTTP_SECRET": "123456secret" 
    }
}
```

将上述Marathon APP的JSON定义保存到一个JSON文件（如`docker-registry.json`），通过调用DCOS的WEB API或者通过DCOS CLI将服务部署到DCOS集群。

WEB API

`curl -i -H "Content-type: application/json" -X POST http://192.168.1.69:8080/v2/apps --data @docker-registry.json`

或 CLI

`dcos marathon app add docker-registry.json`

### 在集群内部访问容器仓库

如果前述私有容器仓库搭建过程中采用的是VIP模式，则在集群内部访问容器仓库时，需要使用VIP：“192.168.0.1”。

`curl -k https://192.168.0.1/v2/_catalog`

在Dockerfile中设置基础镜像时，也需要进行调整：

`FROM 192.168.0.1/base/docker-tomcat-base`

另外，除了前述提到的DnD需要映射主机的`/etc/docker/certs.d`来使用私有容器的签名证书外，对于一些由服务动态创建的容器也需要添加映射配置，如Jenkins动态创建的Slave容器主机。否则，可能会碰到类似`x509: certificate signed by unknown authority`的异常。

### 在集群外部访问容器仓库

因为DCOS集群内部使用VIP“192.168.0.1”来访问容器仓库，VIP“192.168.0.1”对集群外部环境来说是无法访问的，那么集群外部如何访问该私有容器仓库？

从集群外部访问容器仓库有两种途径。

第一种方案：注意到前述Registry的JSON定义中，我们为APP设置了值为**5000**的`servicePort`并且添加了`HAPROXY_GROUP`标签，如果DCOS集群中在**Public** Agent节点上部署了Marathon-LB服务，则可以通过MLB访问容器仓库。

假设Public Agent节点的IP地址为192.168.1.50，则

`curl -k https://192.168.1.50:5000/v2/_catalog`

可以向仓库推送容器镜像：

`docker push 192.168.1.50:5000/base/docker-tomcat-base`

在Dockerfile中设置基础镜像时，同样需要进行调整：

`FROM 192.168.1.50：5000/base/docker-tomcat-base`

第二种方案是为部署的Registry服务设置固定的hostPort（如5000），并固定到一个Agent节点（如：192.168.1.70）上，则可以用与MLB方案相同的方式访问容器仓库。

推荐使用MLB方案。

### 参考

https:\/\/dcos.io\/docs\/1.9\/usage\/tutorials\/registry\/

