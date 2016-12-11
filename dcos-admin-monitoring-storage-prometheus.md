## Prometheus

[Prometheus](https://github.com/prometheus)是一个最初在[SoundCloud](http://soundcloud.com/)上构建的开源系统监控和告警工具包。Prometheus于2016年加入了[Cloud Native Computing Foundation](https://cncf.io/)，作为继Kubernetes之后的第二个加入托管的项目。

### 特性

多维数据模型（由度量名称和键/值对标识的时间序列）

提供灵活的查询语言来利用这些维度

单个服务器节点是自治的，不依赖分布式存储

通过HTTP上的PULL模型进行时间序列数据收集

通过中间网关（Pushgateway）支持PUSH模型收集时间序列数据

通过服务发现或静态配置发现采集的目标（Target）

支持多种模式的图形和仪表板


### 组件

Prometheus生态系统由多个组件组成，其中许多组件是可选的：

Prometheus Server作为核心组件，负责采集和存储时间序列数据

客户端库可用于开发检测应用程序代码

推送网关（Pushgateway）组件用于支持短期任务（short-lived jobs）

基于Rails/SQL实现的GUI仪表板构建器

多种指特定的标信息采集器（exporters），如：HAProxy，StatsD，Ganglia等

告警管理器组件，支持多种方式报警

提供命令行查询工具

其它众多支持工具

### 架构

