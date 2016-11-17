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

#### 部署Registry到DCOS集群

```json
{ 
    "id": "/registry", "cpus": 0.5, "mem": 128, "disk": 0, "instances": 1, 
    "constraints": [["hostname","LIKE","192.168.1.80"]], 
    "container": { 
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
    "healthChecks": [ { "protocol": "TCP", "portIndex": 0, "gracePeriodSeconds": 60, "intervalSeconds": 60, "timeoutSeconds": 20, "maxConsecutiveFailures": 3, "ignoreHttp1xx": false } ], 
    "labels": {"HAPROXY_GROUP": "external"}, 
    "env": { "REGISTRY_HTTP_TLS_CERTIFICATE": "/certs/domain.crt", "REGISTRY_HTTP_TLS_KEY": "/certs/domain.key", "REGISTRY_HTTP_SECRET": "123456secret" }
}
```

修改Jenkins Slave镜像

添加`VOLUME "/etc/docker"`,让动态创建的Jenkins Slave可以使用Agent主机上配置的私有仓库的SSL证书信息。

解决问题：`x509: certificate signed by unknown authority`

修改Jenkins Slave主机配置

### 参考

https:\/\/dcos.io\/docs\/1.9\/usage\/tutorials\/registry\/

