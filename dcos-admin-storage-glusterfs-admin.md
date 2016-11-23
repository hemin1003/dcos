## GlusterFS日常维护

查看GlusterFS中所有的volume

删除GlusterFS磁盘

注： 删除 磁盘 以后，必须删除 磁盘\( `/opt/gluster/data` \) 中的 （ `.glusterfs/` 和`.trashcan/` ）目录。

否则创建新 volume 相同的 磁盘 会出现文件 不分布，或者 类型 错乱 的问题。

卸载某个节点GlusterFS磁盘

设置访问限制,按照每个volume 来限制

添加GlusterFS节点

注：如果是复制卷或者条带卷，则每次添加的Brick数必须是replica或者stripe的整数倍

配置卷



缩容volume

先将数据迁移到其它可用的Brick，迁移结束后才将该Brick移除

在执行了start之后，可以使用status命令查看移除进度

不进行数据迁移，直接删除该Brick

注意，如果是复制卷或者条带卷，则每次移除的Brick数必须是replica或者stripe的整数倍



扩容：



修复命令



迁移volume



均衡volume

