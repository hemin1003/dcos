Deploy tomcat and a war to DCOS

准备

Tomcat下载地址

http:\/\/mirrors.aliyun.com\/apache\/tomcat\/tomcat-8\/v8.5.6\/bin\/apache-tomcat-8.5.6.tar.gz

War下载地址（本地Nexus仓库）

http:\/\/192.168.1.54:8081\/nexus\/content\/repositories\/releases\/com\/example\/example\/1.0\/example-1.0.war



定义marathon服务配置

方案一：直接由marathon启动tomcat并部署war

定义如下json设置，并保存到marathon-tomcat-war.json

```
{
  "id": "tomcat",
  "cmd": "mv *.war apache-tomcat-*/webapps && cd apache-tomcat-* && sed \"s/8080/$PORT/g\" < ./conf/server.xml > ./conf/server-mesos.xml && ./bin/catalina.sh run -config ./conf/server-mesos.xml",
  "mem": 512,
  "cpus": 1.0,
  "instances": 1,
  "uris": [
    "http://wwwftp.ciril.fr/pub/apache/tomcat/tomcat-8/v8.0.20/bin/apache-tomcat-8.0.20.tar.gz",
    "http://192.168.1.54:8081/nexus/service/local/repositories/releases/content/com/example/example/1.0/example-1.0.war"
  ]
}
```

方案二：将Tomcat和War包打包成Docker镜像，部署镜像到DCOS

假设docker镜像名为“`docker-tomcat-war`”，定义如下json配置，并保存到docker-tomcat-war.json

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

