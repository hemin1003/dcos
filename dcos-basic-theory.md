## 系统基础

### CURL

DC/OS系统自带了CURL，而且DC/OS系统内部大量使用了CURL。因此理解CURL的功能和作用是必要的。在访问HTTPS目标地址时，CURL默认会校验服务器证书。DC/OS中CURL使用的证书存放位置位于：

```
/opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem
```

参考：[CURL常用命令](https://gist.github.com/303182519/132568fd0e58cae57202)

### 分布式一致性

### paxos，raft，

### cap理论

### gossip protocol

### 7层网络



