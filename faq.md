#### Firewalld服务未提前关闭，导致安装本地Universe失败

在安装本地Universe时，因之前的安装firewalld未关闭，导致docker引擎无法正常启动。

错误信息：

> `ERROR: COMMAND_FAILED: '/sbin/iptables -w2 -t nat -C POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE' failed: iptables: No chain/target/match by that name.`

解决方案：

重新启动firewalld

> `$ systemctl restart firewalld`

重新启动docker服务

> `$ systemctl restart docker`



