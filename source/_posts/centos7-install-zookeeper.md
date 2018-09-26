---

title: zookeeper安装：单机模式、伪集群模式、集群模式
date: 2018-09-23 13:34:39
tags: [zookeeper,vagrant,]
categories: zookeeper


---

zookeeper的安装分为三种模式： 单机模式、伪集群模式和集群模式。 

安装需要用到的zookeeper文件，到[http://zookeeper.apache.org/](http://zookeeper.apache.org/)通过download链接下载。 

一、单机模式安装zookeeper
1、设置环境变量

``` bash 
$ ZK_FILE=/vagrant/zookeeper-3.4.13.tar.gz
$ ZK_INSTALL_PATH=/opt/zk
$ ZK_INSTALL_DIR=$ZK_INSTALL_PATH/zookeeper-3.4.13
$ ZK_DATA_DIR=$ZK_INSTALL_PATH/zookeeper-3.4.13/data 
$ ZK_LOG_DIR=$ZK_INSTALL_PATH/zookeeper-3.4.13/log
$ ZK_CFG=$ZK_INSTALL_DIR/conf/zoo.cfg 
$ ZK_SERVER_ID=1
$ ZK_PORT=2181
$ ZK_SERVER_LIST=server.1=127.0.0.1:2888:3888
```

![prepare-install-env      ][1] 

2、创建安装目录并将zk到解压安装目录

```  bash
$ mkdir -p $ZK_INSTALL_PATH	
$ tar -zxvf $ZK_FILE -C $ZK_INSTALL_PATH >> /var/null
$ ls -l $ZK_INSTALL_PATH
```

![make-dir-and-uzip        ][2]


3、新建数据和日志目录，生成myid文件，拷贝配置zoo.cfg文件
```  bash
$ mkdir $ZK_DATA_DIR
$ mkdir $ZK_LOG_DIR
$ echo $ZK_SERVER_ID >> $ZK_DATA_DIR/myid
$ cp $ZK_INSTALL_DIR/conf/zoo_sample.cfg $ZK_CFG
$ cat $ZK_CFG
```

![copy-zoo-cfg             ][3] 

4、配置zoo.cfg数据目录、日志目录、端口号、服务器列表并确认配置
```  bash
$ sed -i 's#dataDir=/tmp/zookeeper#dataDir='$ZK_DATA_DIR'#' $ZK_CFG
$ sed -i '$a \dataLogDir='$ZK_LOG_DIR  $ZK_CFG
$ sed -i 's/2181/'$ZK_PORT'/g' $ZK_CFG
$ sed -i '$a \'$ZK_SERVER_LIST $ZK_CFG

$ cat $ZK_CFG
```

![config-zoo-cfg           ][4] 

5、启动zk并查看状态 
```  bash
$ sh $ZK_INSTALL_DIR/bin/zkServer.sh start 
$ sh $ZK_INSTALL_DIR/bin/zkServer.sh status
```
![start-zookeeper          ][5]   

6、利用zkCli操作zk 
```  bash
$ sh $ZK_INSTALL_DIR/bin/zkCli.sh
```
![use-zk-cli               ][6]

可以输入以下命令进行操作, 查看结果
```  bash
> ls / 
> create /test value1
> get /test
```

![zk-cli-command           ][7] 


二、通过vagrant一键安装单机版的zookeeper
1、整理一中的shell命令成zk-install.sh脚本文件
``` bash 
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>zk压缩包路径>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
echo $ZK_FILE
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>zk安装路径&安装目录>>>>>>>>>>>>>>>>>>>>>"
echo $ZK_INSTALL_PATH
echo $ZK_INSTALL_DIR
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>zk数据目录&日志目录>>>>>>>>>>>>>>>>>>>>>"
echo $ZK_DATA_DIR
echo $ZK_LOG_DIR
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>zk的myid>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
echo $ZK_SERVER_ID
echo ">>>>>>>>>>>>>>>>>>>>>>>>>zk端口及服务器列表>>>>>>>>>>>>>>>>>>>>>>>>>"
echo $ZK_PORT
echo $ZK_SERVER_LIST

echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>zk配置文件>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
ZK_CFG=$ZK_INSTALL_DIR/conf/zoo.cfg
echo $ZK_CFG

echo ">>>>>>>>>>>>>>>>>>>>新建zk安装目录并解压zk到该目录>>>>>>>>>>>>>>>>>>>" 
mkdir -p $ZK_INSTALL_PATH	

tar -zxvf $ZK_FILE -C $ZK_INSTALL_PATH >> /var/null


echo ">>>>>>>>>>>>>>>>>>>>>新建zk数据和日志目录及myid>>>>>>>>>>>>>>>>>>>>>" 
mkdir $ZK_DATA_DIR
mkdir $ZK_LOG_DIR
echo $ZK_SERVER_ID >> $ZK_DATA_DIR/myid

echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>配置zoo.cfg>>>>>>>>>>>>>>>>>>>>>>>>>>>>>" 
cp $ZK_INSTALL_DIR/conf/zoo_sample.cfg $ZK_CFG

sed -i 's#dataDir=/tmp/zookeeper#dataDir='$ZK_DATA_DIR'#' $ZK_CFG
sed -i '$a \dataLogDir='$ZK_LOG_DIR  $ZK_CFG
sed -i 's/2181/'$ZK_PORT'/g' $ZK_CFG
sed -i '$a \'$ZK_SERVER_LIST $ZK_CFG

tail -n 20 $ZK_CFG
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>启动zk>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>" 	
sh $ZK_INSTALL_DIR/bin/zkServer.sh start 
```

2、新建Vagrantfile文件 
``` ruby 
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	#设置虚拟机的box
	config.vm.box = "centos7-base-jdk8"

	
	#设置虚拟机的主机名
	config.vm.hostname = "centos7-jdk8-zk-standalone"
	#设置ip
	config.vm.network "private_network", ip: "192.168.13.10"
	#Virtualbox相关配置
	config.vm.provider "virtualbox" do |v|
		#设置虚拟机的名称
		v.name = "centos7-jdk8-zk-standalone"
		
		#设置虚拟机的内存大小为2G
		v.memory = 2048 
		
		#设置虚拟机的CPU个数
		v.cpus = 2 
	end 

	#使用shell脚本进行软件安装和配置
	config.vm.provision "shell" do |s| 
		s.path = "zk-install.sh"
		s.env = {ZK_FILE: "/vagrant/zookeeper-3.4.13.tar.gz",
			ZK_INSTALL_PATH: "/opt/zk", 
			ZK_INSTALL_DIR: "/opt/zk/zookeeper-3.4.13", 
			ZK_PORT: "2181",  
			ZK_DATA_DIR: "/opt/zk/zookeeper-3.4.13/data", 
			ZK_LOG_DIR: "/opt/zk/zookeeper-3.4.13/log", 
			ZK_SERVER_ID: "1", 
			ZK_SERVER_LIST: "server.1=127.0.0.1:2888:3888"}
	end
end
```
3、通过vagrant up启动虚拟机   

![vagrant-up-standardlone-1][8] 

![vagrant-up-standardlone-2][9]  

三、通过vagrant一键安装zookeeper伪集群
1、zk伪集群指的是在一台集群上安装zk集群
需要注意每个zk进程对外端口（默认2181，用于客户端链接zk集群），选主端口（默认3888-进行leader选举时使用的端口），集群通信端口（默认2888-集群follow链接leader的通信端口）配置不冲突。 
myid=1,  2181, 2888, 3888
myid=2,  2182, 2887, 3887
myid=3,  2183, 2886, 3886

2、zk-install.sh同二使用的是相同内容，Vagrantfile如下： 
``` ruby 

# -*- mode: ruby -*-
# vi: set ft=ruby :
	
Vagrant.configure("2") do |config|
	#设置虚拟机的box
	config.vm.box = "centos7-base-jdk8"
	
	#设置虚拟机的主机名
	config.vm.hostname = "centos7-jdk-zk-local-cluster"
	
	#Virtualbox相关配置
	config.vm.provider "virtualbox" do |v|
		#设置虚拟机的名称
		v.name = "centos7-jdk-zk-local-cluster"
		
		#设置虚拟机的内存大小为1G
		v.memory = 2048 
		
		#设置虚拟机的CPU个数
		v.cpus = 2 
	end 
	

	(1..3).each do |i| 
	#使用shell脚本进行软件安装和配置
		config.vm.provision "shell" do |s| 
			s.path = "zk-install.sh"
			s.env = {ZK_FILE: "/vagrant/zookeeper-3.4.13.tar.gz",
				ZK_INSTALL_PATH: "/opt/zk/zk-server#{i}", 
				ZK_INSTALL_DIR: "/opt/zk/zk-server#{i}/zookeeper-3.4.13", 
				ZK_PORT: "218#{i}",  
				ZK_DATA_DIR: "/opt/zk/zk-server#{i}/zookeeper-3.4.13/data", 
				ZK_LOG_DIR: "/opt/zk/zk-server#{i}/zookeeper-3.4.13/log", 
				ZK_SERVER_ID: "#{i}", 
				ZK_SERVER_LIST: "server.1=127.0.0.1:2888:3888\\nserver.2=127.0.0.1:2887:3887\\nserver.3=127.0.0.1:2886:3886"}
		end 
    end 		
end
```   
3、同样使用vagrant up 命令，可以启动虚拟机，并自动安装 
4、ssh进入虚拟机， 查看各个目录下对应的zk安装情况
可以看到，zk-server2作为leader节点， 其他两个节点作为follower节点

![zk-local-cluster-status  ][10] 

5、可以通过xshell给三个节点的zkCli.sh 分别发送命令，

![zk-local-cluster-cli     ][11]  


四、通过vagrant一键安装zookeeper集群
1、集群ip规划  
zk_server1 =>   1  192.168.13.21  centos7-jdk-zk-cluster-1
zk_server2 =>   2  192.168.13.22  centos7-jdk-zk-cluster-2 
zk_server3 =>   3  192.168.13.23  centos7-jdk-zk-cluster-3

2、zk-install.sh同二使用的是相同内容，Vagrantfile如下： 
``` ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	#设置虚拟机的box
	config.vm.box = "centos7-base-jdk8"
	
	zk_servers = {
		:zk_server1 => ["192.168.13.21", "1", "centos7-jdk-zk-cluster-1"], 
		:zk_server2 => ["192.168.13.22", "2", "centos7-jdk-zk-cluster-2"], 
		:zk_server3 => ["192.168.13.23", "3", "centos7-jdk-zk-cluster-3"]
	}
	zk_server_list = "server.1=192.168.13.21:2888:3888\\nserver.2=192.168.13.22:2888:3888\\nserver.3=192.168.13.23:2888:3888"
	
	zk_servers.each do |zk_server_name, zk_server_cfg| 
		config.vm.define zk_server_name do |app_config| 
			
			#设置虚拟机的主机名
			app_config.vm.hostname = zk_server_cfg[2]
			#设置ip
			app_config.vm.network "private_network", ip: zk_server_cfg[0]
			#Virtualbox相关配置
			app_config.vm.provider "virtualbox" do |v|
				#设置虚拟机的名称
				v.name = zk_server_cfg[2]
				
				#设置虚拟机的内存大小为1G
				v.memory = 1048 
				
				#设置虚拟机的CPU个数
				v.cpus = 2 
			end 

			#使用shell脚本进行软件安装和配置
			app_config.vm.provision "shell" do |s| 
				s.path = "zk-install.sh"
				s.env = {ZK_FILE: "/vagrant/zookeeper-3.4.13.tar.gz",
					ZK_INSTALL_PATH: "/opt/zk", 
					ZK_INSTALL_DIR: "/opt/zk/zookeeper-3.4.13", 
					ZK_PORT: "2181",  
					ZK_DATA_DIR: "/opt/zk/zookeeper-3.4.13/data", 
					ZK_LOG_DIR: "/opt/zk/zookeeper-3.4.13/log", 
					ZK_SERVER_ID: zk_server_cfg[1], 
					ZK_SERVER_LIST: zk_server_list}
			end
		end 
	end 
end
```
3、通过vagrant up 拉起三台虚拟机，并自动安装zk
安装完毕后，进入虚拟机，确认是否安装正确

![zk-cluster-status        ][12]

OK, 到这里就结束了， 三种模式的安装通过vagrant可以快速安装实验环境。 


  [1]: centos7-install-zookeeper/prepare-install-env.png
  [2]: centos7-install-zookeeper/make-dir-and-uzip.png
  [3]: centos7-install-zookeeper/copy-zoo-cfg.png
  [4]: centos7-install-zookeeper/config-zoo-cfg.png
  [5]: centos7-install-zookeeper/start-zookeeper.png
  [6]: centos7-install-zookeeper/use-zk-cli.png
  [7]: centos7-install-zookeeper/zk-cli-command.png
  [8]: centos7-install-zookeeper/vagrant-up-standardlone-1.png
  [9]: centos7-install-zookeeper/vagrant-up-standardlone-2.png
  [10]: centos7-install-zookeeper/zk-local-cluster-status.png
  [11]: centos7-install-zookeeper/zk-local-cluster-cli.png
  [12]: centos7-install-zookeeper/zk-cluster-status.png 
