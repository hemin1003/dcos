## 持续集成与部署（CI\/CD）

Blue-Green部署 [Blue Green Deployment by Martin Fowler](http://martinfowler.com/bliki/BlueGreenDeployment.html)

使用Maven和Jenkins为不同的环境（**开发、测试和生产环境**）打包

配置Maven构建环境

参考

https:\/\/www.zybuluo.com\/haokuixi\/note\/25985

https:\/\/maven.apache.org\/guides\/mini\/guide-building-for-different-environments.html

根据环境不同，选择不同的配置文件

http:\/\/stackoverflow.com\/questions\/9912632\/maven-reading-a-property-from-an-external-properties-file

在Jenkins中使用Config File Provider插件提供不同的配置文件

http:\/\/stackoverflow.com\/questions\/30692093\/config-file-management

