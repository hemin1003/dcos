## 理解Exhibitor

### 共享配置存储

为了确保服务的可靠性，Zookeeper通常以集群的方式部署运行，而Exhibitor的核心特性是能够与每个运行的实例共享配置。为了达到这个目标，Exhibitor实现了**Amazon的S3**，**共享网络文件**和**单独的Zookeeper集群**三种共享配置存储方案。在实际部署时可根据需要选择三种方案中的一种。

当通过Exhibitor的Web管理界面对配置进行更改时，其他实例将监控到这些更改并据此采取行动。如果需要，每个ZooKeeper实例将被停止，zoo.cfg文件被重建，ZooKeeper服务被重新启动。

S3

--configtype参数的值为“s3”。以下是允许Exhibitor与S3存储桶通信的IAM策略示例：

```json
{
  "Statement": [
    {
      "Sid": "Stmt1386687089728",
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:DeleteObject",
        "s3:GetBucketAcl",
        "s3:GetBucketPolicy",
        "s3:GetObject",
        "s3:GetObjectAcl",
        "s3:ListBucket",
        "s3:ListBucketMultipartUploads",
        "s3:ListMultipartUploadParts",
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::bucket-name/*",
        "arn:aws:s3:::bucket-name"
      ]
    }
  ]
}
```
