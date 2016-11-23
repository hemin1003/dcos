## 系统基础

### CURL

DCOS系统自带了CURL，而且DCOS系统内部大量使用了CURL。因此理解CURL的功能和作用是必要的。在访问HTTPS目标地址时，CURL默认会校验服务器证书。DCOS中CURL使用的证书存放位置位于：

```
/opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem
```

分布式一致性

paxos，raft，

cap理论

gossip protocol

7层网络

