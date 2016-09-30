### 在Centos7.2上安装Docker

#### 1.更新Centos到最新版本

> `$ sudo yum upgrade --assumeyes --tolerant`
> 
> `$ sudo yum update --assumeyes`

#### 2.检查内核版本&gt;3.10

> `$ uname -r`
> 
> `3.10.0-327.10.1.el7.x86_64`

#### 3.启用OverlayFS

> `$ sudo tee /etc/modules-load.d/overlay.conf <<-'EOF'`
> 
> `overlay`
> 
> `EOF`

#### 4.重启系统

> `$ reboot`

#### 5.检查OverlayFS是否启用

> `$ lsmod | grep overlay`
> 
> `overlay`

#### 6.配置Docker的yum仓库

> `$ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'`
> 
> `[dockerrepo]`
> 
> `name=Docker Repository`
> 
> `baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/`
> 
> `enabled=1`
> 
> `gpgcheck=1`
> 
> `gpgkey=https://yum.dockerproject.org/gpg`
> 
> `EOF`

#### 7.配置systemd服务在OverlayFS上运行Docker Daemon

> `$ sudo mkdir -p /etc/systemd/system/docker.service.d && sudo tee /etc/systemd/system/docker.service.d/override.conf <<- EOF`
> 
> `[Service]`
> 
> `ExecStart=`
> 
> `ExecStart=/usr/bin/docker daemon --storage-driver=overlay -H fd://`
> 
> `EOF`

#### 8.安装Docker engine，daemon和service

> `$ sudo yum install -y docker-engine-1.11.2`
> 
> `$ sudo systemctl start docker`
> 
> `$ sudo systemctl enable docker`

#### 9.完成，检查Docker是否正确安装

> `$ sudo docker ps`

