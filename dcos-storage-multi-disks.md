## 磁盘资源

Mesos提供了一种同时管理多个磁盘资源的机制。在创建[持久化卷](/dcos-storage-persistent-volume.md)时，框架可以根据提供的磁盘资源的`source`字段决定是否使用该资源。

### 磁盘资源类别

Mesos目前支持三种`Disk`磁盘资源，包括：

* **Root**磁盘类型，在`DiskInfo`中不包含`source`集合定义的磁盘

* **Path**磁盘类型，在`DiskInfo`中的`source`定义里含有`PATH`枚举集合定义，该类型描述里同时包含一个`root`定义了存储数据的目录位置

* **Mount**磁盘类型，在`DiskInfo`中的`source`定义里含有`MOUNT`枚举集合定义，该类型描述里同时包含一个`root`定义了存储数据的安装点（mount point）


在Agent节点启动时，可以通过--resources参数传递一个JSON格式的磁盘资源定义，为Agent提供上述几种不同的磁盘资源。默认情况（未指定--resources）下，Agent节点仅将Root磁盘资源提供给DCOS集群。

提示：一旦为Agent节点手动指定了任何Disk磁盘资源，Mesos将停止自动监测Root磁盘资源。因此，想要使用Root磁盘资源，必须在资源定义中进行定义。

### Root磁盘

Root磁盘是Mesos中的基本磁盘资源。通常它是指启动该Agent的操作系统的磁盘空间。使用Root磁盘时，数据会保存在Agent的work\_dir参数指定的存储位置。Root磁盘定义示例如下：

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

为Agent节点设置磁盘资源时，可以给一个该磁盘资源指定一个role，当进行资源分配时，可以在指定了role的磁盘资源上进行资源静态预留（statically reserving）。

### Path磁盘

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

Mount磁盘

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

Block磁盘

