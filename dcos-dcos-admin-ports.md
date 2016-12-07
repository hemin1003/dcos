## DC/OS内部端口

以下是DC/OS内部服务使用的端口及其相应的systemd unit列表。了解这些端口占用，便于在集群出现异常时进行故障排除。如，碰到53端口被占用（通常是被dnsmasq占用），DC/OS的Spartan服务可能无法正常启动；无法访问Master节点上的8080端口时，可能是Master节点上的Marathon服务出现异常。

### 所有节点

#### TCP

* 61003: dcos-rexray \(default\)

* 61053: dcos-mesos-dns

* 61420: dcos-epmd

* 61421: dcos-minuteman

* 62053: dcos-spartan

* 62080: dcos-navstar

* 62501: dcos-spartan

* 62502: dcos-navstar

* 62503: dcos-minuteman


#### UDP

* 62053: dcos-spartan

* 64000: dcos-navstar


### Master

#### TCP

* 53: dcos-spartan

* 80: dcos-adminrouter

* 443: dcos-adminrouter

* 1050: dcos-3dt

* 1801: dcos-oauth

* 2181: dcos-exhibitor

* 5050: dcos-mesos-master

* 7070: dcos-cosmos

* 8080: dcos-marathon

* 8123: dcos-mesos-dns

* 8181: dcos-exhibitor

* 9990: dcos-cosmos

* 15055: dcos-history-service


#### UDP

* 53: dcos-spartan

### Agent, Public Agent

#### TCP

* 5051: dcos-mesos-slave
* 61001: dcos-adminrouter-agent


### 参考



