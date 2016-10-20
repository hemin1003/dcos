Deploy tomcat and a war to DCOS

### 准备工作

**Tomcat下载地址**

http:\/\/mirrors.aliyun.com\/apache\/tomcat\/tomcat-8\/v8.5.6\/bin\/apache-tomcat-8.5.6.tar.gz

**War下载地址（本地Nexus仓库）**

http:\/\/192.168.1.54:8081\/nexus\/content\/repositories\/releases\/com\/example\/example\/1.0\/example-1.0.war

### 定义marathon服务配置

**方案一：直接由marathon启动tomcat并部署war**

定义如下json设置，并保存到app.json

```
{
  "id": "tomcat",
  "cmd": "mv *.war apache-tomcat-*/webapps && cd apache-tomcat-* && sed \"s/8080/$PORT/g\" < ./conf/server.xml > ./conf/server-mesos.xml && ./bin/catalina.sh run -config ./conf/server-mesos.xml",
  "mem": 512,
  "cpus": 1.0,
  "instances": 1,
  "uris": [
    "http://mirrors.aliyun.com/apache/tomcat/tomcat-8/v8.5.6/bin/apache-tomcat-8.5.6.tar.gz",
    "http://192.168.1.54:8081/nexus/content/repositories/releases/com/example/example/1.0/example-1.0.war"
  ]
}
```

**方案二：将Tomcat和War包打包成Docker镜像，部署镜像到DCOS**

假设docker镜像名为“`docker-tomcat-war`”，定义如下json配置，并保存到app.json

```
{
 "id": "exampleapp",
 "instances": 1,
 "cpus": 0.25,
 "mem": 128,
 "uris":[],
 "env": {  },
 "ports":[9000],
 "container": {
   "type": "DOCKER",
   "volumes": [],
   "docker": {
     "image": "chrisrc/docker-tomcat-war:0.0.1",
     "network": "BRIDGE",
     "portMappings": [{ "containerPort": 8080, "servicePort": 9000 , "hostPort": 0, "protocol": "tcp" }]
    }
  }
}
```

**调用DC\/OS的REST接口部署APP**

根据上述两种方案，选择其中的一种，执行下述指令部署应用：

```
curl -X POST -H "Content-Type:application/json" "http://192.168.1.69:8080/v2/apps" -d @/Users/chrisrc/Dcos/Services/app.json
```

### 为服务添加心跳监测

可以为要部署的Tomcat应用增加服务心跳监测，同样存在两种配置（可以任选其一）：

```
"healthChecks": [
    {
      "protocol": "HTTP",
      "portIndex": 0,
      "path": "/",
      "gracePeriodSeconds": 5,
      "intervalSeconds": 20,
      "maxConsecutiveFailures": 3
    }
]
```

或者

```
"healthChecks": [
    {
      "protocol": "COMMAND",
      "command": { "value": "curl -f -X GET http://$HOST:$PORT" },
      "gracePeriodSeconds": 5,
      "intervalSeconds": 20,
      "maxConsecutiveFailures": 3
    }
]
```

完整的配置示例如下

```
{
 "id": "tomcat",
 "cmd": "mv *.war apache-tomcat-*/webapps && cd apache-tomcat-* && sed \"s/8080/$PORT/g\" < ./conf/server.xml > ./conf/server-mesos.xml && ./bin/catalina.sh run -config ./conf/server-mesos.xml",
 "mem": 512,
 "cpus": 1.0,
 "instances": 1,
 "uris": [
 "http://mirrors.aliyun.com/apache/tomcat/tomcat-8/v8.5.6/bin/apache-tomcat-8.5.6.tar.gz",
 "http://192.168.1.54:8081/nexus/content/repositories/releases/com/example/example/1.0/example-1.0.war"
 ],
 "healthChecks": [
   {
     "protocol": "HTTP",
     "portIndex": 0,
     "path": "/",
     "gracePeriodSeconds": 5,
     "intervalSeconds": 20,
     "maxConsecutiveFailures": 3
   }
 ]
}
```

补充：如果要在jenkins直接使用脚本配置，则可以如下定义脚本：

```
curl -i -H "Content-type: application/json" -X POST http://192.168.1.69:8080/v2/apps -d '
{
 "id": "tomcat",
 "cmd": "mv *.war apache-tomcat-*/webapps && cd apache-tomcat-* && sed \"s/8080/$PORT/g\" < ./conf/server.xml > ./conf/server-mesos.xml && ./bin/catalina.sh run -config ./conf/server-mesos.xml",
 "mem": 512,
 "cpus": 1.0,
 "instances": 1,
 "uris": [
 "http://mirrors.aliyun.com/apache/tomcat/tomcat-8/v8.5.6/bin/apache-tomcat-8.5.6.tar.gz",
 "http://192.168.1.54:8081/nexus/content/repositories/releases/com/example/example/1.0/example-1.0.war"
 ],
 "healthChecks": [
 {
 "protocol": "HTTP",
 "portIndex": 0,
 "path": "/",
 "gracePeriodSeconds": 5,
 "intervalSeconds": 20,
 "maxConsecutiveFailures": 3
 }
 ]
}
'
```

