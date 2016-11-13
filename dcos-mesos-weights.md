## 权重（Weights）

在Mesos中，权重可以用于控制提供给不同角色的集群资源的相对份额。

Mesos在0.28.0（含）之前，权重只能在启动Mesos的Master节点时通过指定--weights命令行参数来配置。如果一个角色未特别指定权重，则默认值为1.0。如果不更配置并重新启动所有Mesos Master节点，则动态更改角色的权重。

Mesos自1.0版本开始，包含一个\/weights管理接口，允许在运行时更改权重。命令行参数--weights已弃用。

### 权重管理接口

权重管理接口http:\/\/&lt;Master-IP&gt;:5050\/weights提供了一种REST风格的管理接口，支持如下操作：

* 通过PUT指令更新权重。

* 通过GET指令查询当前设置的权重。


更新权重

定义一个JSON文件，设置角色的权重。

```json
[ 
    { 
        "role": "role1", 
        "weight": 2.0 
    }, 
    { 
        "role": "role2", 
        "weight": 3.5 
    } 
]
```

调用权重更新接口，更新权重

```
$ curl -d @weights.json -X PUT http://<master-ip>:<port>/weights
```

如果Master配置了显式的角色白名单，那么，请求中的角色必须都在角色白名单中存在。

权重现在在集群引导和任何更新后都会在注册表中保存。一旦权重被保留在注册表中，任何随后以--weights指定的方式启动的Mesos的Master节点都会发出警告并改用注册表中的权重值。

调用权重更细节看的响应值为下列之一：

* 200 OK: Success \(the update request was successful\)._ _
* 400 BadRequest: Invalid arguments \(e.g., invalid JSON, non-positive weights\). 
* 401 Unauthorized: Unauthenticated request.\* 
* 403 Forbidden: Unauthorized request.

### 查询权重设置

调用以下指令查询权重

```
$ curl -X GET http://<master-ip>:<port>/weights
```

该指令的响应如下：

```json
[ 
    { 
        "role": "role2", 
        "weight": 3.5 
    }, 
    { 
        "role": "role1", 
        "weight": 2.0 
    } 
]
```

调用权重更细节看的响应值为下列之一：

* 200 OK: Success.\* 
* 401 Unauthorized: Unauthenticated request.

