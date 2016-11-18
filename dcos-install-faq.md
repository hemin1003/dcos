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

