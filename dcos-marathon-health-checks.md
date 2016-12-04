### 应用健康检查

#### 健康检查

在DC/OS中，可以为每一个应用设置独立的健康检查。默认情况下，Mesos判定应用是否“健康”的标准是任务的状态为TASK\_RUNNING。Marathon在应用的任务资源定义中设置了一个健康检查项，可以通过调用REST API为应用设置健康检查。

健康检查通过的必要条件包括：

* HTTP响应返回代码为200至399（含399）之间。

* 响应在timeoutSeconds参数设定的时间间隔内返回。


如果应用健康检查失败，而且超过了参数maxConsecutiveFailures的设定值，这个任务会被杀死。

在生产环境中，默认的配置选项不能使用，根据应用的类型，镜像的大小，以及每次发布的时间，经过多次测试，可以选出恰当的时间来进行设定。也可以根据其他因素对配置参数进行调整，避免容器没有启动就被kill掉。

**注意：**如果应用未通过健康检查，且应用通过Marathon-LB绑定了外部服务端口，则Marathon-LB不会正确发布此服务。可以通过下述URL查看MLB端口绑定是否正常：

```
http://<public-node-ip>:9090/haproxy?stats
```

### **示例配置**

#### HTTP

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

#### Mesos HTTP

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

#### HTTPS

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

注意：HTTPS检查不会校验SSL证书

#### TCP

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

TCP检查的源代码如下：

```
def tcp( 
 task: Task, 
 check: MarathonTcpHealthCheck, 
 host: String, port: Int): Future[Option[HealthResult]] = { 
  val address = s"$host:$port" 
  val timeoutMillis = check.timeout.toMillis.toInt 
  log.debug("Checking the health of [{}] via TCP", address)

  Future {     
   val address = new InetSocketAddress(host, port)     
   val socket = new Socket     
   scala.concurrent.blocking {     
    socket.connect(address, timeoutMillis)     
    socket.close() 
   } 
  Some(Healthy(task.taskId, task.runSpecVersion, Timestamp.now())) 
}(ThreadPoolContext.ioContext) }
```

#### COMMAND

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

注意：如果在command中需要使用双引号“"”，则必须对双引号进行转义，因为Mesos通过`/bin/sh -c ""指令调用健康检查的command。`

```
{ "protocol": "COMMAND", "command": { "value": "/bin/bash -c \\\"</dev/tcp/$HOST/$PORT0\\\"" } }
```

### **参数定义**

健康检查的配置项参数定义如下:

|  | 参数 | 取值 | 说明 |
| :--- | :--- | :--- | :--- |
|  | **protocol** | 默认值：HTTP，备选值：HTTP / HTTPS/ TCP / COMMAND / MESOS\_HTTP / MESOS\_HTTPS / MESOS\_TCP | HTTP/HTTPS/TCP检查是通过当前的Marathon Leader 来执行确认； MESOS\_HTTP/MESOS\_HTTPS/MESOS\_TCP和COMMAND是由正在执行当前任务的节点检查确认 |
|  | **gracePeriodSeconds** | 可选，默认值：300 | 在此设定值阶段内，所有检查失败将被忽略，或者直至任务第一次进入健康状态即TASK\_RUNNING |
|  | **intervalSeconds** | 可选，默认值：60 | 两次健康检查操作的时间间隔 |
|  | **maxConsecutiveFailures** | 可选，默认值：3 | 在任务被杀死之前健康检查连续失败的次数。对HTTP和TCP检查，如果设置为0，则任务即使健康检查失败也不会被杀死。 |
|  | **timeoutSeconds** | 可选，默认值：20 | 不管有没有响应返回值，健康检查操作的返回超出此设定值即认为检查未通过。 |
|  | **portIndex** | 可选，默认值：0 | 健康检查所访问的此应用定义中的`ports或portDefinitions数组中的端口索引。使用索引可以让APP使用自定义端口` |
|  | **port** | 可选，默认值：无 | 健康检查所访问的端口。注意：_portIndex或port均未设置时，默认使用portIndex；如果同时设置，则优先使用port参数的设置值_ |
|  | **path** | 可选，默认值：“\/” | 该参数仅适用于MESOS\_HTTP,MESOS\_HTTPS,HTTP,HTTPS，用于设置健康检查访问的路径 |
| **ignoreHttp1x** | 可选，默认值：false | 该参数仅适用于HTTP,HTTPS，忽略检查响应返回的100-199状态码，当返回值位于这个区间时，保持前一次的健康状态不变。 | \#\#\# **应用服务健康状态生命周期** |

应用服务的健康状态可以用一个有限状态机表示，如下图所示：  
**i** 请求的应用服务实例数  
**r** 运行的应用服务实例数  
**h** 健康的应用服务实例数

![](/assets/dcos_marathon_app_state.png)

### 任务终止

#### **TASK\_KILLING**

TASK\_KILLING是任务的一个状态，意味着该任务收到一个Kill请求，当前正处于宽限期。其它工具如负载均衡或服务发现不应再将请求路由到该任务。  
**taskKillGracePeriodSeconds**  
尽管健康检查可以让你确定一个应用服务不健康时可以kill掉它，taskKillGracePeriodSeconds允许设置一个值，这个值定义了executor向任务发送SIGTERM消息通知任务停止到executor再次发送SIGKILL消息正式杀掉该任务之间的时间间隔。这个参数在任务不会立即停止，而是需要一个退出时间时非常有用。如果没有设置此参数，其默认值为3秒钟。

如何手动管理任务，请参考DC/OS CLI及管理UI相关章节。

