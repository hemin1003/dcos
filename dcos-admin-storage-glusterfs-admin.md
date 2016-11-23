## GlusterFS日常维护

查看GlusterFS中所有的Volume

删除GlusterFS磁盘

注意：删除磁盘以后，必须删除磁盘\( `/opt/gluster/data` \) 中的（ `.glusterfs/` 和`.trashcan/` ）目录。

否则创建新Volume相同的磁盘会出现文件不分布，或者类型错乱的问题。

卸载某个节点上的GlusterFS磁盘

设置访问限制,按照每个Volume来限制

添加GlusterFS节点

注意：如果是复制卷或者条带卷，则每次添加的Brick数必须是replica或者stripe的整数倍

配置卷

缩容volume

先将数据迁移到其它可用的Brick，迁移结束后才将该Brick移除

在执行了start之后，可以使用status命令查看移除进度

不进行数据迁移，直接删除该Brick

注意，如果是复制卷或者条带卷，则每次移除的Brick数必须是replica或者stripe的整数倍

扩容

修复命令

迁移Volume

均衡Volume

### 参考



