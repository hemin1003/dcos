## 备份集群安装文件

在完成DC/OS集群安装后，除非明确规划保留Bootstrap节点，否则建议立即备份DC/OS的安装文件。这些安装文件在后续可用于增加新的（私有或公有）Agent节点。

在Bootstrap节点上，进入`genconf/serve`目录，执行下面的命令生成备份包：

```
$ cd genconf/serve
$ sudo tar cf dcos-install.tar *
```

备份生成的`dcos-install.tar`文件到备份主机。



