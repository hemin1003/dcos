## HDFS on DCOS

### 安装HDFS

安装要求：

dcos package install hdfs

查看HDFS的连接信息

通过WEB接口：

`http://192.168.1.69/service/hdfs/v1/connection/hdfs-site.xml`

通过DCOS CLI：

dcos hdfs connection hdfs-site.xml

