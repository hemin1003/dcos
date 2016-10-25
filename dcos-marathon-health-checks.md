Health Checks and Task Termination

在DCOS中，可以为每一个应用设置独立的健康检查。默认情况下，Mesos判定应用是否“健康”的标准是任务的状态为TASK\_RUNNING。Marathon在应用的任务资源定义中设置了一个健康检查项，可以通过调用REST API为应用设置健康检查。

健康检查通过的必要条件包括：

HTTP响应返回代码为200至399（含399）之间。

HTTP响应在timeoutSeconds参数设定的时间间隔内返回。

如果应用健康检查失败，而且超过了参数maxConsecutiveFailures的设定值，这个任务会被杀死。

示例

HTTP

```
{ 
  "path": "/api/health", 
  "portIndex": 0, 
  "protocol": "HTTP", 
  "gracePeriodSeconds": 300, 
  "intervalSeconds": 60, 
  "timeoutSeconds": 20, 
  "maxConsecutiveFailures": 3, 
  "ignoreHttp1xx": false 
}
```

Mesos HTTP

```
{ 
  "path": "/api/health", 
  "portIndex": 0, 
  "protocol": "MESOS_HTTP", 
  "gracePeriodSeconds": 300, 
  "intervalSeconds": 60, 
  "timeoutSeconds": 20, 
  "maxConsecutiveFailures": 3 
}
```

secure HTTP

```
{ 
  "path": "/api/health", 
  "portIndex": 0, 
  "protocol": "HTTPS", 
  "gracePeriodSeconds": 300, 
  "intervalSeconds": 60, 
  "timeoutSeconds": 20, 
  "maxConsecutiveFailures": 3, 
  "ignoreHttp1xx": false
}
```

TCP

```
{ 
  "portIndex": 0, 
  "protocol": "TCP", 
  "gracePeriodSeconds": 300, 
  "intervalSeconds": 60, 
  "timeoutSeconds": 20, 
  "maxConsecutiveFailures": 0
}
```

COMMAND

```
{ 
  "protocol": "COMMAND", 
  "command": { "value": "curl -f -X GET http://$HOST:$PORT0/health" }, 
  "gracePeriodSeconds": 300, 
  "intervalSeconds": 60, 
  "timeoutSeconds": 20, 
  "maxConsecutiveFailures": 3
}
```


