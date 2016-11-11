# Summary

## DC\/OS之导论
* [导论](README.md)
    * [DCOS概览](dcos概览.md)
    * [基础理论](dcos-basic-theory.md)
* [环境搭建](环境搭建.md)
    * [安装环境准备](dcos-install-default.md)
        * [在Centos上安装Docker](dcos-install-docker-on-centos.md)
    * [GUI安装](dcos-install-by-gui.md)
    * [通过CLI安装](dcos-install-by-cli.md)
    * [高级安装](dcos-install-by-advanced-mode.md)
* [集群维护](dcos-install-maintainance.md)
    * [备份集群安装文件](dcos-install-backup-installer-file.md)
    * [集群节点管理](dcos-install-nodes-management.md)
    * [集群卸载](dcos-install-m-uninstall-all.md)

## DC\/OS之原理
* [资源调度策略](dcos-theory-schedule-strategy.md)
* [系统服务](dcos-system-components.md)
* [Exhibitor服务](dcos-component-exhibitor.md)
* [Reservation](dcos-mesos-reservation.md)
* [Authorization](dcos-mesos-authorization.md)
* [存储策略与方案](dcos-storage.md)
    * [存储驱动](存储驱动.md)
    * [磁盘资源](dcos-storage-multi-disks.md)
        * [增加磁盘资源](dcos-storage-mount-disk-resources.md)
        * [增加NFS存储](dcos-storage-nfs-server.md)
    * [持久化卷](dcos-storage-persistent-volume.md)
        * [本地持久化卷](dcos-storage-persistent-volume-local.md)
        * [外部持久化卷](外部持久化卷.md)
* [服务发现与负载均衡](服务发现与负载均衡.md)
    * [VIPs](vips.md)
    * [Marathon-LB](marathon-lb.md)
    * [Mesos-DNS](mesos-dns.md)
    * [Spartan](spartan.md)
* [容器网络方案](dcos-network.md)
    * [服务端口配置](dcos-network-marathon-ports.md)
    * [使用VIPs](dcos-network-vips.md)
    * [基于VIPs的负载调度](dcos-network-vips-lb.md)
* [Mesos](dcos-mesos.md)
* [Marathon](dcos-marathon.md)
    * [Health Checks and Task Termination](dcos-marathon-health-checks.md)
    * [容器运行管理](dcos-marathon-container.md)
* [Universe](dcos-component-universe.md)
    * [应用的管理](dcos-component-universe-usage.md)

## DC\/OS之服务
* [HDFS on DCOS](hdfs-on-jenkins.md)
* [Jenkins on DCOS](jenkins-on-dcos.md)
    * [自定义Jenkins Slave容器镜像](dcos-service-jenkins-custom-dind.md)
    * [示例：在Jenkins on DCOS上编译部署Tomcat应用](deploy.md)
* [FAQ](faq.md)
* [部署Cassandra到DCOS](部署cassandra到dc.md)
* [Marathon-LB](marathon-lb.md)
    * [为Marathon-LB启用SSL](dcos-service-marathon-lb-ssl.md)
* [Docker Registry](docker-registry.md)
* [DCOS 应用仓库之Universe](dcos-应用仓库之universe.md)
* [DCOS应用市场之应用仓库的管理](dcos应用市场之应用仓库的管理.md)

## DCOS之管理
* [DCOS CLI](dcos-admin-cli.md)
* [DCOS 管理UI](dcos-admin-ui.md)

## DC\/OS之应用
* [持续集成与持续部署](dcos-usage-cicd.md)
    * [应用打包与镜像构建](dcos-usage-cicd-build-by-maven.md)
    * [持续集成与构建](dcos-usage-cicd-integration.md)
    * [持续部署](dcos-usage-cicd-deploy.md)

## DCOS之开发

