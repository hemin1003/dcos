资源与属性

Mesos有两种基本语义来描述组成集群的Agent节点：即资源（Resources）和属性（Attributes）。其中资源由Mesos的Master节点管理，另一种属性只是简单地传递给使用集群的框架，由框架根据属性值进行逻辑处理。

类型

资源和属性通过**值（Value）**来描述，这些值是有类型（Types）的。Mesos中由资源和属性共同支持的值的类型包括：scalar，ranges，sets和text。这些类型的定义如下：

```
scalar : floatValue

floatValue : ( intValue ( "." intValue )? ) | ...

intValue : [0-9]+

range : "[" rangeValue ( "," rangeValue )* "]"

rangeValue : scalar "-" scalar

set : "{" text ( "," text )* "}"

text : [a-zA-Z0-9_/.-]
```

属性
属性是由键值对组成的描述信息，其中属性的值是可选的。Mesos在向框架提供资源时顺带将资源的属性提交给框架。属性的值支持三种类型：scalar，range和text。

```
attributes : attribute ( ";" attribute )*

attribute : text ":" ( scalar | range | text )
```

资源
Mesos负责管理三种不同的资源类型：scalars，ranges和sets。这三种资源类型用于描述Agent节点所能提供的各种资源，如scalar资源类型可以用来描述Agent节点上的内存数量。Scalar资源类型的值支持小数（如1.5CPUs），小数位的精度精确到小数点后3位，如1.512CPUs。对于GPU资源，Mesos仅支持整数值。
资源可以通过一个JSON数组或者一个由分号分隔的键值对字符串来定义。对于JSON的格式，可参考**protobuf**消息定义（位于`include/mesos/mesos.proto`）中`Resource`部分的描述。

```json
[ { "name": "<resource_name>", "type": "SCALAR", "scalar": { "value": <resource_value> } }, { "name": "<resource_name>", "type": "RANGES", "ranges": { "range": [ { "begin": <range_beginning>, "end": <range_ending> }, ... ] } }, { "name": "<resource_name>", "type": "SET", "set": { "item": [ "<first_item>", ... ] }, "role": "<role_name>" }, ...]
```

键值对字符串的定义：

```
resources : resource ( ";" resource )*

resource : key ":" ( scalar | range | set )

key : text ( "(" resourceRole ")" )?

resourceRole : text | "*"
```

注意，resourceRole必须是一个有效的值，具体定义参考后续章节。

预定义的用途和约定

下述是几种具有预定义行为的资源：

cpus

gpus

disk

mem

ports

注意，`disk`和`mem`资源通过兆字节（M）进行计量，Mesos的用户界面会将这些资源值转换为用户友好的格式，如15000将显示为14.65GB。

如果一个Agent节点没有cpus和mem资源，那么这个Agent节点不会向框架公布任何资源。

资源示例

默认情况下，Mesos会在Agent节点上的mesos-agent进程（DCOS中为dcos-mesos-agent.service\/dcos-mesos-agent-public.service服务）启动时自动侦测节点上可用的资源。或者，你可以明确指明Agent节点提供了哪些资源。

下述示例明确指明了Agent节点的资源：

```
--resources='cpus:24;gpus:2;mem:24576;disk:409600;ports:[21000-24000,30000-34000];bugs(debug_role):{a,b,c}'

--resources='[{"name":"cpus","type":"SCALAR","scalar":{"value":24}},{"name":"gpus","type":"SCALAR","scalar":{"value":2}},{"name":"mem","type":"SCALAR","scalar":{"value":24576}},{"name":"disk","type":"SCALAR","scalar":{"value":409600}},{"name":"ports","type":"RANGES","ranges":{"range":[{"begin":21000,"end":24000},{"begin":30000,"end":34000}]}},{"name":"bugs","type":"SET","set":{"item":["a","b","c"]},"role":"debug_role"}]'
```
或者也可以在一个resource.txt文件中进行定义：

```json
[ { "name": "cpus", "type": "SCALAR", "scalar": { "value": 24 } }, { "name": "gpus", "type": "SCALAR", "scalar": { "value": 2 } }, { "name": "mem", "type": "SCALAR", "scalar": { "value": 24576 } }, { "name": "disk", "type": "SCALAR", "scalar": { "value": 409600 } }, { "name": "ports", "type": "RANGES", "ranges": { "range": [ { "begin": 21000, "end": 24000 }, { "begin": 30000, "end": 34000 } ] } }, { "name": "bugs", "type": "SET", "set": { "item": [ "a", "b", "c" ] }, "role": "debug_role" }]
```
然后传递给mesos-agent进程
```
$ path/to/mesos-agent --resources=file:///path/to/resources.txt ...
```
在上述示例中定义了三种不同资源类型的6种资源，即scalars类型的cpus，gpus，mem和disk，一个range类型的ports和一个set类型的bugs。bugs指定了角色debug_role，其它资源没有指定角色因此属于默认角色。注意，“默认角色”可以通过--default_role来指定。
scalar类型的cpus，值为24
scalar类型的gpus，值为2
scalar类型的mem，值为24576
scalar类型的disk，值为409600
range类似的ports，值在21000~24000和30000~34000两个区间
set类型的bugs，值为a，b和c，并且分配给debug_role角色

可以通过mesos-agent的命令行参数--attributes设置Agent节点的属性：

```
--attributes='rack:abc;zone:west;os:centos5;level:10;keys:[1000-1500]'
```
示例中的Agent节点包含5个属性：
rack，  具有text类型的值abc
zone，  具有text类型的值west
os，    具有text类型的值centos5
level， 具有scalar类型的值10
keys，  具有range类型的值1000~1500区间

