## DC/OS集群节点管理

### 新增节点

#### 新增Master节点

当前版本（1.8）DC/OS在集群部署完成后不支持添加新的Master节点，因此，在构建集群时，必须根据业务规划提前确定好Master节点的数量，推荐至少3个节点，如果集群规模足够庞大，则可以增至5个节点。

#### 新增Agent节点

**准备工作**

1. 确保DC/OS集群在第一次完成安装时，[备份了集群安装文件](/dcos-install-backup-installer-file.md)。

2. 确保执行了DC/OS集群[安装环境准备](/dcos-install-default.md)工作中所有涉及Agent节点的操作。

3. 将Bootstrap节点上的SSH公钥拷贝到新增Agent节点主机的`/root/.ssh/authorized_keys文件。`


**操作步骤**

* 使用scp将集群安装文件拷贝的新增Agent节点主机
  ```
  scp dcos-install.tar  <username>@<new-agent-node-ip>:~/dcos-install.tar
  ```


* 登录到新增节点主机
  ```
  ssh <username>@<new-agent-node-ip>
  ```


* 在新增节点上创建DCOS临时安装目录
  ```
  sudo mkdir -p /opt/dcos_install_tmp
  ```


* 解压上传的集群安装文件
  ```
  sudo tar -xf dcos-install.tar -C /opt/dcos_install_tmp
  ```


* 执行下述命令添加节点
  添加私有Agent节点：

  ```
  sudo bash /opt/dcos_install_tmp/dcos_install.sh slave
  ```

  添加公有Agent节点：

  ```
  sudo bash /opt/dcos_install_tmp/dcos_install.sh slave_public
  ```


* 通过下述两个命令可以分别查询当前集群中私有节点和公有节点的数量
  私有：

  ```
  dcos node --json | jq --raw-output '.[] | select(.reserved_resources.slave_public == null) | .id' | wc -l
  ```

  公有：

  ```
  dcos node --json | jq --raw-output '.[] | select(.reserved_resources.slave_public != null) | .id' | wc -l
  ```


### 删除节点

可以从当前DC/OS集群中删除单个节点，具体操作步骤如下：

```
sudo -i  /opt/mesosphere/bin/pkgpanda uninstall

sudo reboot

sudo rm -rf /opt/mesosphere /etc/mesosphere
```

如果节点删除过程中出现异常，要手工检查`/etc/systemd/system`目录下是否存在以“dcos”开头的服务，需要手动清除。

如果想完全清除节点上的所有集群相关的数据，需要再执行下述命令清理：

```
sudo rm -rf /var/lib/dcos /var/lib/mesos
```

### 变更节点类型



