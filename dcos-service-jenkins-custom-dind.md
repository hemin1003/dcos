## 自定义Jenkins Slave容器镜像

修改Jenkins Slave镜像

添加`VOLUME "/etc/docker"`,让动态创建的Jenkins Slave可以使用Agent主机上配置的私有仓库的SSL证书信息。

解决问题：`x509: certificate signed by unknown authority`

修改Jenkins Slave主机配置

