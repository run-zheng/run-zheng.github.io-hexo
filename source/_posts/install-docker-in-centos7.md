---

title: Centos7安装Docker
date: 2018-04-15 22:45:05
tags: [docker,centos7,virtualbox,vagrant,vagrant box,]
categories: 折腾过程

---

## 一、安装Docker 

关于docker介绍的文章非常多，这里就不详细介绍了。 从1.13版本开始，docker分为社区版本CE和企业版本EE。版本号采用时间线命名方法。社区版本按照stable和edge两种方式发布， 每个季度发布stable版本。

docker（version 18.03.0-ce）要求CentOS7， linux内核版本>=3.10。

#### 1. 两种方式安装
docker安装比较简单，可以有两种方式进行安装
a. 手动按步骤安装
```
#生成缓存
sudo yum makecache fast

#Delta RPMs disabled because /usr/bin/applydeltarpm not installed  增量rpm包相关
sudo yum provides '*/applydeltarpm'
sudo yum install -y deltarpm

#升级包和内核
sudo yum -y update

#yum-util 提供yum-config-manager功能
sudo yum install -y -q yum-utils

#添加docker的repo
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce
```
b. 在线脚本安装
```
#安装docker
#sudo wget -qO- https://get.docker.com | sh
```
其实，安装方式a跟安装方式b下载的脚本中的安装过程类似，如果有兴趣可以将get.docker.com下载的脚本保存下来，这个脚本兼容各类操作系统的安装。

#### 2. 设置开机启动和启动docker service
```
#设置docker服务启动后自运行 启动docker服务
sudo systemctl enable docker && sudo systemctl start docker
```

#### 3. 开放管理端口和设置mirror

```
#设置mirror
#开放管理端口映射237 5为主管理端口，unix:///var/run/docker.sock用于本地管理，7654是备用的端口
sudo sed -i -e 's!^ExecStart=/usr/bin/dockerd!ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H tcp://0.0.0.0:7654 --registry-mirror=https://docker.mirrors.ustc.edu.cn!' /lib/systemd/system/docker.service
sudo systemctl daemon-reload

sudo echo 'export DOCKER_HOST=tcp://0.0.0.0:2375' >> /etc/profile  
source /etc/profile

sudo systemctl restart docker
```

#### 4. 添加用户(vagrant)到docker权限组

```
#添加vagrant用户添加到docker权限组
sudo usermod -aG docker vagrant
```

#### 5. 安装结果
```
#测试docker安装结果  查看安装版本
docker -v

#拉取hello-world镜像
docker pull hello-world

#运用hello-world容器
docker run hello-world
```

看到以下结果，说明运行成功
![docker-run-hello-world][1]

## 二、vagrant自动安装
如果已经安装了vagrant，可以将上述安装过程自动化，并且安装完成后可以到处box镜像，后续安装docker集群或者创建带docker的虚拟机，都可以很方便操作。
以下脚本基于[制作一个CentOS7的Vagrant Box](https://run-zheng.github.io/2018/04/03/How-To-Create-A-CentOS7-Vagrant-Base-Box/)生成的centos7-base box。 

#### 1. 自动化安装的Vagrantfile
a. 创建目录并保存vagrantfile到目录中
b. 运行vagrant up命令，并耐心等待几分钟，中间需要升级包和内核、安装docker， 过程会比较久，是网速而定。 

====================================[Vagrantfile][3]====================================
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<-SCRIPT
#centos7-base默认配置了 使用163的centos源 
#sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup 
#sudo wget http://mirrors.163.com/.help/CentOS7-Base-163.repo -O /etc/yum.repos.d/CentOS-Base.repo


#======================安装方式1============================
#生成缓存
sudo yum makecache fast 
#Delta RPMs disabled because /usr/bin/applydeltarpm not installed  增量rpm包相关
sudo yum provides '*/applydeltarpm'
sudo yum install -y deltarpm 
#升级包和内核
sudo yum -y update

#yum-util 提供yum-config-manager功能
sudo yum install -y -q yum-utils 
#添加docker的repo
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce

#======================安装方式2=================================
#安装docker 
#sudo wget -qO- https://get.docker.com | sh 
#======================安装完成==================================

#设置docker服务启动后自运行 启动docker服务
sudo systemctl enable docker && sudo systemctl start docker

#设置mirror
#开放管理端口映射237 5为主管理端口，unix:///var/run/docker.sock用于本地管理，7654是备用的端口
sudo echo "====set docker manager port and mirror start===="
sudo sed -i -e 's!^ExecStart=/usr/bin/dockerd!ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H tcp://0.0.0.0:7654 --registry-mirror=https://docker.mirrors.ustc.edu.cn!' /lib/systemd/system/docker.service
sudo echo "====set docker manager port and mirror end====" && sudo systemctl daemon-reload

sudo echo 'export DOCKER_HOST=tcp://0.0.0.0:2375' >> /etc/profile  
sudo echo "====docker host====" && source /etc/profile

sudo systemctl restart docker

#添加vagrant用户添加到docker权限组
sudo usermod -aG docker vagrant 

#默认的公钥在启动的时候会被移除（在vagrant up的时候可以看到下面的提示信息），如果需要自己生产公私钥，去掉这一行
#default: Vagrant insecure key detected. Vagrant will automatically replace
#default: this with a newly generated keypair for better security.
#default:
#default: Inserting generated public key within guest...
#default: Removing insecure key from the guest if it's present...
#default: Key inserted! Disconnecting and reconnecting using new SSH key...
curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys

#测试docker安装结果  查看安装版本
docker -v
#拉取hello-world镜像
docker pull hello-world
#运用hello-world容器
docker run hello-world

#清理包
sudo yum clean all
SCRIPT

Vagrant.configure("2") do |config|
	#设置虚拟机的box
	config.vm.box = "centos7-base"
	
	#设置虚拟机的主机名
	config.vm.hostname = "centos7-base-docker"
	
	#Virtualbox相关配置
	config.vm.provider "virtualbox" do |v|
		#设置虚拟机的名称
		v.name = "centos7-base-docker"
		
		#设置虚拟机的内存大小为2G
		v.memory = 2048 
		
		#设置虚拟机的CPU个数
		v.cpus = 2 
	end 
	
	#使用shell脚本进行软件安装和配置
	config.vm.provision "shell", inline: $script 
	
end
```

#### 2. 运行结果
看到以下结果，说明安装成功
![after install][2]

#### 3. 创建centos7-base-docker的box文件
```
#运行 package命令， 生成box文件
vagrant package --output centos7-base-docker.box --base centos7-base-docker
```
#### 4. 测试box文件的可用性 
首先将生成的box添加到box list中， 然后用vagrant init当前目录， vagrant up拉起虚拟机，完成后，vagrant ssh链接进入虚拟机就可以操作了。

```
vagrant box add centos7-base-docker centos7-base-docker.box
vagrant box list
vagrant init centos7-base-docker
vagrant up
vagrant ssh
```

  [1]: install-docker-in-centos7/docker-run-hello-world.png
  [2]: install-docker-in-centos7/after_install.png
  [3]: Vagrantfile