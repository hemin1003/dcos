#### Firewalld服务未提前关闭，导致安装本地Universe失败

在安装本地Universe时，因之前的安装firewalld未关闭，导致docker引擎无法正常启动。

错误信息：

> `ERROR: COMMAND_FAILED: '/sbin/iptables -w2 -t nat -C POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE' failed: iptables: No chain/target/match by that name.`

解决方案：

重新启动firewalld

> `$ systemctl restart firewalld`

修改`/etc/systemd/system/docker.service` 或 `/etc/systemd/system/docker.service.d/docker.conf` 文件（根据具体安装略有不同） 添加如下配置:

`--iptables=false`

然后重启docker服务：

`systemctl daemon-reload`

`service docker restart`

#### 安装过程中，待安装的服务一直处于Staging状态

可能的原因有两个：一个是无法连接到Universe，一个是系统报：Insufficient resource ...。

前者是因为Universe配置的问题，例如Mesos Agent无法访问5000端口，可参考上一个问题；

后者是因为slave集群内没有private agent或者private agent的资源不够。待部署的Service设置的资源Role，默认为“**\***”，需要设置为：“**slave\_public**”。

#### Centos 7上Docker镜像运行错误

[Docker run fails with "invalid argument" when using overlay driver on top of xfs](https://github.com/docker/docker/issues/10294)

解决方案：升级Linux内核至3.18.4+

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml
```

