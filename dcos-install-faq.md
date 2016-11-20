## FAQ

### Bootstrap节点是否必须？

在安装参数“”未设置成“”时，Bootstrap节点不是必须的。可以在安装完成后，将该节点纳入集群作为Agent节点。但是该节点在安装过程中生成集群的安装包，需要提前做好备份，该安装包可用于增加新的Agent节点。

### Master节点需要部署几个？

开发测试环境中，1个master即可，master节点故障仅对新的服务部署及现有环境配置修改产生影响，不对当前正在运行的集群及服务造成影响。如果想确保开发测试环境的可靠性，推荐3个节点。

生成环境中，根据规模，3个节点或5个节点即可。Master节点数不是越多越好，Master节点数量与集群性能成反比。

### Firewalld服务未提前关闭，导致安装本地Universe失败

在安装本地Universe时，因之前的安装firewalld未关闭，导致docker引擎无法正常启动。

错误信息：

```
ERROR: COMMAND_FAILED: '/sbin/iptables -w2 -t nat -C POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE' failed: iptables: No chain/target/match by that name.
```

解决方案：

重新启动firewalld

```
$ systemctl restart firewalld
```

修改`/etc/systemd/system/docker.service` 或 `/etc/systemd/system/docker.service.d/docker.conf` 文件（根据具体安装略有不同） 添加如下配置:

```
--iptables=false
```

然后重启docker服务：

```
systemctl daemon-reload
service docker restart
```

### 安装过程中，待安装的服务一直处于Staging状态

可能的原因有两个：一个是无法连接到Universe，一个是系统报：Insufficient resource ...。

前者是因为Universe配置的问题，例如Mesos Agent无法访问5000端口，可参考上一个问题；

后者是因为slave集群内没有private agent或者private agent的资源不够。待部署的Service设置的资源Role，默认为“**\***”，需要设置为：“**slave\_public**”。

### Centos 7上Docker镜像运行错误

[Docker run fails with "invalid argument" when using overlay driver on top of xfs](https://github.com/docker/docker/issues/10294)

解决方案：升级Linux内核至3.18.4

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

yum --enablerepo=elrepo-kernel install kernel-ml
```

### DC\/OS系统出现问题的排查思路

0.Ensure that you have a valid `ip-detect` script and working DNS resolvers in your `config.yaml` :point_left: 95% of all installation issues are caused by errors in these two configuration items You can manually run _`ip-detect`_ on all the nodes in your cluster and\/or check _`/opt/mesosphere/bin/detect_ip`_ on an existing installation to ensure that it returns a valid IP address i.e., an IP address without extra new lines or white-spaces or special\/hidden characters, with which to bind the DC\/OS services. Please try to copy or base your _`ip-detect`_ on the examples listed in _[_https:\/\/dcos.io\/docs\/1.8\/administration\/installing\/custom\/advanced\/_](https://dcos.io/docs/1.8/administration/installing/custom/advanced/)_ w.r.t DNS resolution: \_Ideally_, forward _and_ reverse lookups for FQDNs, Short Hostnames and IP addresses should work, but we can run DC\/OS in environments without valid DNS support but the following _must_ work \(esp. for Spark and some other frameworks\), regardless: `hostname -f` _must_ return the FQDN `hostname -s` _must_ return the Short Hostname Check the output of `hostnamectl` for sanity on all your nodes as well
In general, this is the sequence in which we will have to troubleshoot:_\(Exhibitor -&gt; Mesos Master -&gt; Mesos-DNS -&gt; Spartan -&gt; {Marathon, Metronome, Adminrouter}\)_ and checking that all services are up and healthy on the Masters before moving to the Agents

1.Ensure that firewalls or whatever connection-filtering mechanism is in use is not interfering with cluster component communications. TCP, UDP _and_ ICMP need to be permitted.

2.Check to see if Exhibitor has come up and has settled `http://<PICK_A_MASTER_IP>:8181/exhibitor` otherwise check the Exhibitor service logs: `journalctl -flu dcos-exhibitor`Please ensure that `/tmp` is mounted _without_ `noexec` otherwise Exhibitor will fail to bring up Zookeeper because Java JNI won't be able to `exec` a file it creates in `/tmp` with `permission denied` errors in the Exhibitor service log file: `journalctl -flu dcos-exhibitor`To fix: `mount -o remount,exec /tmp`Check the output of `/exhibitor/v1/cluster/status` and ensure that it has the correct number of masters that you expect and that _all of them_ are `"serving"` _but only one_ of them has `"isLeader": true`

e.g., While SSH'ed into a Master node:

```
curl -fsSL http://localhost:8181/exhibitor/v1/cluster/status | python -m json.tool
[ 
    { "code": 3, "description": "serving", "hostname": "10.0.6.70", "isLeader": false }, 
    { "code": 3, "description": "serving", "hostname": "10.0.6.69", "isLeader": false }, 
    { "code": 3, "description": "serving", "hostname": "10.0.6.68", "isLeader": true }
]
```

Note: You _may_ have to wait a while \(~10-15 minutes\) for this to happen in multi-master setups. If it doesn't converge after this much time has elapsed you may need to look through `journalctl -flu dcos-exhibitor` logs with a fine-toothed comb.

3.Assuming Exhibitor is up, check if you can ping `ready.spartan` otherwise check the Spartan service logs: `journalctl -flu dcos-spartan`

4.Assuming Spartan is up and you can ping `ready.spartan` check if you can ping `leader.mesos` and `master.mesos` otherwise check the Mesos-DNS service logs: `journalctl -flu dcos-mesos-dns`

5.If you can ping `ready.spartan` but not `leader.mesos` you may also have to check the Mesos master\(s\) service logs: `journalctl -flu dcos-mesos-master` because the Mesos master needs to be up and a leader must be elected before Mesos-DNS can generate its DNS records from `/state` \(

