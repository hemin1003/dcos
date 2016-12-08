## 磁盘资源

Mesos提供了一种同时管理多个磁盘资源的机制。在创建[持久化卷](/dcos-storage-persistent-volume.md)时，框架可以根据提供的磁盘资源的`source`字段决定是否使用该资源。

### 磁盘资源类别

Mesos目前支持三种`Disk`磁盘资源，包括：

* **Root**磁盘类型，在`DiskInfo`中不包含`source`集合定义的磁盘

* **Path**磁盘类型，在`DiskInfo`中的`source`定义里含有`PATH`枚举集合定义，该类型描述里同时包含一个`root`定义了存储数据的目录位置

* **Mount**磁盘类型，在`DiskInfo`中的`source`定义里含有`MOUNT`枚举集合定义，该类型描述里同时包含一个`root`定义了存储数据的挂载点（mount point）


在Agent节点启动时，可以通过--resources参数传递一个JSON格式的磁盘资源定义，为Agent提供上述几种不同的磁盘资源。默认情况（未指定--resources）下，Agent节点仅将Root磁盘资源提供给DCOS集群。

提示：一旦为Agent节点手动指定了任何Disk磁盘资源，Mesos将停止自动监测Root磁盘资源。因此，想要使用Root磁盘资源，必须在资源定义中进行定义。

#### Root磁盘

Root磁盘是Mesos中的基本磁盘资源。通常它是指启动该Agent的操作系统的磁盘空间。使用Root磁盘时，数据会保存在Agent的`work_dir`参数指定的存储位置(在DC/OS中默认为`/var/lib/dcos/mesos-resources`)。Root磁盘定义示例如下：

```json
{ 
    "resources": [ 
        { 
            "name": "disk", 
            "type": "SCALAR", 
            "scalar": { "value": 2048 } 
        } 
    ]
}
```

为Agent节点设置磁盘资源时，可以给一个磁盘资源指定一个role，当进行资源分配时，可以在指定了role的磁盘资源上进行资源静态预留（statically reserving）。

#### Path磁盘

Path磁盘是一种辅助类型的磁盘资源。它通过将磁盘分割成小块的方式，在其上创建使用小于磁盘上的总可用空间的持久卷。Path磁盘通常用于额外的日志存储空间，文件归档存储，缓存或者直接用于一些非性能关键的应用。通过在`DiskInfo`的`source`标签下指定`root`指向所创建目录的路径，即可将该路径下的存储资源提交给Agent节点。

```json
{ 
    "resources": [ 
        { 
            "name": "disk", 
            "type": "SCALAR", 
            "scalar": { "value": 2048 }, 
            "disk": { "source": { "type": "PATH", "path": { "root": "/mnt/data" } } } 
        } 
    ]
}
```

可以在操作系统磁盘空间下创建多个目录，并把这些目录挂载到Path磁盘资源下，但是这种用法最好仅限于测试环境。在同一个操作系统下创建多个Path磁盘资源时，需要对磁盘资源（空闲空间）进行静态分区。例如，在`/foo`下面挂载了10G的存储空间，Agent节点配置了两个Path磁盘分别为`/foo/disk1`和`/foo/disk2`。为避免空间资源不足的情况发生，Agent节点启动时，disk1和disk2需要进行特定配置，两者总的存储空间之和不应超过10G。

同上，可以给一个磁盘资源指定一个role，当进行资源分配时，可以在指定了role的磁盘资源上进行资源静态预留（statically reserving）。

#### Mount磁盘

Mount磁盘是一种辅助类型的磁盘资源。它不能被应用框架切割为小的存储块。这种方式牺牲了灵活性，但可以确保应用框架能够控制整个磁盘空间。通常用于数据库存储，预写日志或其他性能关键的应用。

在Linux操作系统下，Mount磁盘必须映射为`/proc/mounts`表中的一个挂载点。

Mount磁盘除了性能优势之外，运行在这些磁盘之上的应用可以依靠磁盘空间不足时的磁盘错误进行故障处理。基于这种预期，Mesos禁用了对Mount磁盘资源的`disk/du`隔离。

```json
{ 
    "resources": [ 
        { 
            "name": "disk", 
            "type": "SCALAR", 
            "scalar": { "value": 2048 }, 
            "disk": { "source": { "type": "MOUNT", "mount": { "root": "/mnt/data" } } } 
        } 
    ]
}
```

#### Block磁盘

Mesos当前不支持直接暴露原始的块设备。

### 实现机制

Path磁盘会在root目录下创建不同的子目录用于区分在其上创建的不同的持久化卷。当Path磁盘上的一个持久化卷被销毁时，Mesos会删除该卷下的所有文件和目录，同时包括Mesos为该卷所创建的子目录。

Mount磁盘不会额外创建子目录，允许应用使用整个磁盘空间。该结构允许Mesos任务直接访问已存在目录结构的持久化卷，这对于简化提取已存在的数据（如Postgres数据库或HDFS数据目录）非常有用。当Mount磁盘上的一个持久化卷被销毁时，Mesos会删除该卷下的所有文件和目录，但不会删除root目录即挂载点。

了解这些实现机制，有助于运维过程中检查或清除残余数据。

### DC/OS控制台中的磁盘信息

![](/assets/dcos-disk-resources-ex.png)

上图是通过DC/OS的WEB管理控制台显示的Agent节点的资源信息，其中DISK为24.9GB，这是上述磁盘类型的资源总和。当时磁盘消耗占比为10%左右。

可以通过该节点的`/var/lib/dcos/mesos-resources`文件中的配置信息查看磁盘空间的来源。

