## Universe

### 搭建本地Universe仓库

编译并构建仓库镜像

1. 克隆Universe源码到本地：

  ```
  git clone https:\/\/github.com\/mesosphere\/universe.git
  ```

  确认当前工作分支位于`version-3.x`上。

2. 确保**jsonschema**工具在命令行环境中可用：

  ```
  sudo pip install jsonschema
  ```

3. 在Universe目录下执行：

  ```
  scripts/build.sh
  ```

  确认仓库编译成功，此时在`docker/server/target`目录下生成了仓库的索引文件。

4. 按照下述命令生成Universe的容器镜像文件

```
  DOCKER\_IMAGE="chrisrc\/universe-server" DOCKER\_TAG="0.0.1" docker\/server\/build.bash


或

DOCKER\_IMAGE="192.168.1.51:5000\/chrisrc\/universe-server" DOCKER\_TAG="0.0.1" docker\/server\/build.bash

```
5. 执行下述命令将Universe镜像推送到Docker Hub（或私有仓库）

```
DOCKER\_IMAGE="chrisrc\/universe-server" DOCKER\_TAG="0.0.1" docker\/server\/build.bash publish
```


    部署本地Universe到DCOS
    在上述编译构建过程中的第3步，生成Universe索引文件的同时，在`docker/server/target`目录下也生成了一个marathon.json文件。

```
{ "id": "/universe", "instances": 1, "cpus": 0.25, "mem": 128, "requirePorts": true, "container": { "type": "DOCKER", "docker": { "network": "BRIDGE", "image": "192.168.1.51:5000/chrisrc/universe-server:0.0.1", "portMappings": [   {     "containerPort": 80,     "hostPort": 8085,     "protocol": "tcp"   } ] }, "volumes": [] }, "healthChecks": [ { "gracePeriodSeconds": 120, "intervalSeconds": 30, "maxConsecutiveFailures": 3, "path": "/repo-empty-v3.json", "portIndex": 0, "protocol": "HTTP", "timeoutSeconds": 5 } ], "constraints": [ [ "hostname", "UNIQUE" ] ]}
```

