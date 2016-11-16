## 搭建私有容器仓库

搭建私有容器仓库

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

修改Jenkins Slave主机配置

### 参考

https:\/\/dcos.io\/docs\/1.9\/usage\/tutorials\/registry\/

