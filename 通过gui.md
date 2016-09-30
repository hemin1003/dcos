### 通过GUI安装DC\/OS

#### 1.DC\/OS安装文件下载就绪后，在Bootstrap节点上执行如下命令启动安装：

> `$ sudo bash dcos_generate_config.sh --web`

如果想查看调试输出，可以添加-v参数

> `$ sudo bash dcos_generate_config.sh --web -v`

#### 2.安装程序启动后，终端会有类似如下日志显示：

> `Running mesosphere/dcos-genconf docker with BUILD_DIR set to /home/centos/genconf`
> 
> `16:36:09 dcos_installer.action_lib.prettyprint:: ====> Starting DC/OS installer in web mode`
> 
> `16:36:09 root:: Starting server ('0.0.0.0', 9000)`

#### 3.通过浏览器访问DC\/OS安装界面：**http:\/\/&lt;bootstrap-node-public-ip&gt;:9000**

#### 4.点击“**Begin Installation**”开始安装



