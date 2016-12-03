## 约束

约束能够控制应用在哪个（些）Agent节点上运行，因此既可以用来优化容错（将应用部署到多个节点），也可以用来归集（让所有应用运行在同一个节点）。约束包括三个部分：一个字段名，一个运算符和一个可选的参数字段。字段名可以是Agent节点的主机名hostname或任意属性。

### 字段

#### Hostname字段

hostname字段与Agent节点的主机名相匹配，参考下述UNIQUE运算符查看使用示例。

hostname字段支持Marathon所有的运算符。

#### Attribute字段

如果字段名称不是Agent节点的主机名，它将被视为Agent节点的属性。节点属性可以用来给Agent节点做标记。如果在Age节点上未定义指定的属性，大多数运算符将拒绝在该节点上运行任务。事实上，现在只有UNLIKE运算符符总是会接受这个资源配给来运行任务，而其他运算符将总是拒绝它。

属性字段支持Marathon所有的运算符。

Marathon支持文本，标量，范围和集合属性值。对于标量，范围和集合，Marathon将对格式化的值执行按字符串比较。

对于范围和集合，格式分别为\[begin-end，...\]和{item，...}。例如，可以定义一个格式为\[100-200\]的范围和格式为{a，b，c}的集合。

LIKE和UNLIKE运算符允许正则表达式;要匹配任何值，请使用字符串。

### 运算符

#### UNIQUE

UNIQUE告诉Marathon在所有应用程序任务中强制属性的唯一性。例如，以下约束确保每个主机上只运行一个应用程序任务：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-unique", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["hostname", "UNIQUE"]] 
}'
```

#### CLUSTER

CLUSTER允许所有应用程序任务运行在具有某个属性的一个或多个Agent节点上。例如，如果应用程序具有特殊的硬件需求，或者希望在同一机架上运行它们以实现低延迟：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-cluster", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "CLUSTER", "rack-1"]] 
}'
```

可以使用此运算符和hostname属性将应用程序绑定到特定节点：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-cluster", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["hostname", "CLUSTER", "a.specific.node.com"]] 
}'
```

#### GROUP\_BY

GROUP\_BY可以让应用在机架或数据中心之间均匀分配，以实现高可用性：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-group-by", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "GROUP_BY"]] 
}'
```

Marathon仅区分属性值的不同，而不区分属性值本身。因此，GROUP\_BY接受的参数值代表了在不同属性值的数量。如果不指定该数量，那么所有的任务会分配到节点属性的值均为其中的同一个的这些节点上。例如，如果想将应用部署到3个不同的机架上，则按如下配置：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-group-by", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "GROUP_BY", "3"]] 
}'
```

#### LIKE

LIKE接受正则表达式作为参数，可以让应用仅在字段值与正则表达式匹配的Agent节点上运行：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-group-by", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "LIKE", "rack-[1-3]"]] 
}'
```

注意，第三个参数值是必须的。

#### UNLIKE

与LIKE运算符类似，但仅让任务在字段值与正则表达式不匹配的Agent节点上运行：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-group-by", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "UNLIKE", "rack-[7-9]"]] 
}'
```

#### MAX\_PER

MAX\_PER接受一个数字作为参数，指定每个group的最大值。可用于限制跨机架或数据中心的应用数：

```
curl -X POST -H "Content-type: application/json" localhost:8080/v2/apps -d '{ 
    "id": "sleep-group-by", 
    "cmd": "sleep 60", 
    "instances": 3, 
    "constraints": [["rack_id", "MAX_PER", "2"]] 
}'
```

注意，第三个参数值是必须的。

### 参考

[https://mesosphere.github.io/marathon/docs/constraints.html](https://mesosphere.github.io/marathon/docs/constraints.html)

