---

title: Centos7安装jdk8并制作box镜像
date: 2018-09-19 00:24:51
tags: [jdk8,vagrant,centos7,virtualbox,vagrant box,]
categories: 折腾过程


---


一、centos7安装jdk
1、下载jdk安装文件
[https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

2、解压jdk到安装目录
``` bash 
$ mkdir  /usr/java
$ touch /vagrant/jdk-install.log 
$ tar -zxvf  /vagrant/jdk-8u181-linux-x64.tar.gz  -C /usr/java  >> /vagrant/jdk-install.log 
$ tail -n 10 /vagrant/jdk-install.log 
$ ls -l /usr/java
```
![tar-jdk][1]

3、设置环境变量到/etc/profile 
``` bash 
$ JAVA_HOME=/usr/java/jdk1.8.0_181
$ JAVA_BIN=/usr/java/jdk1.8.0_181/bin

$ echo "export JAVA_HOME="$JAVA_HOME >>/etc/profile
$ echo "export JAVA_BIN="$JAVA_BIN >>/etc/profile
$ echo "export PATH=\$PATH:\$JAVA_BIN" >>/etc/profile
$ echo "export CLASSPATH=.:"\$JAVA_HOME/lib:\$JAVA_HOME/jre/lib >>/etc/profile

$ tail -n 10 /etc/profile
```

![set-etc-profile][2]

4、验证安装
``` bash 
$ source /etc/profile
$ java -version
```

![java-version][3]




二、通过vagrant一键式安装
将一中的脚本执行过程制作成脚本，在vagrantfile中直接引用执行脚本，一键安装
shell脚本jdk-install.sh如下： 
``` bash 
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>jdk压缩包路径>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
echo $JDK_FILE
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>jdk安装路径>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
echo $JDK_INSTALL_PATH

echo ">>>>>>>>>>>>>>>>>>>创建目录并解压jdk压缩包到安装路径>>>>>>>>>>>>>>>>>>"
echo $JDK_INSTALL_DIR
mkdir $JDK_INSTALL_DIR
touch $JDK_INSTALL_DIR/jdk-install.log 
tar -zxvf $JDK_FILE -C $JDK_INSTALL_DIR  >> $JDK_INSTALL_DIR/jdk-install.log 
tail -n 10 $JDK_INSTALL_DIR/jdk-install.log 
 ls -l  $JDK_INSTALL_DIR

echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>设置环境变量>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
JAVA_HOME=$JDK_INSTALL_PATH
JAVA_BIN=$JDK_INSTALL_PATH/bin
echo $JAVA_HOME
echo $JAVA_BIN

echo ">>>>>>>>>>>>>>>>>>>>>>>设置环境变量到/etc/profile>>>>>>>>>>>>>>>>>>>>>>"
echo "export JAVA_HOME="$JAVA_HOME >>/etc/profile
echo "export JAVA_BIN="$JAVA_BIN >>/etc/profile
echo "export PATH=\$PATH:\$JAVA_BIN" >>/etc/profile
echo "export CLASSPATH=.:"\$JAVA_HOME/lib:\$JAVA_HOME/jre/lib >>/etc/profile

tail -n 10 /etc/profile

echo ">>>>>>>>>>>>>>>>>>>>>>>>>>source一下让设置生效>>>>>>>>>>>>>>>>>>>>>>>>>"
source /etc/profile
echo ">>>>>>>>>>>>>>>>>>>>>>>>>>看下java命令是否可用>>>>>>>>>>>>>>>>>>>>>>>>>>"
java -version
javac -version 
echo ">>>默认的公钥在启动的时候会被移除,作为base的box,用默认公钥方便ssh登录>>>"
curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys
```

Vagrantfile内容如下：
``` ruby 
# -*- mode: ruby -*-
# vi: set ft=ruby :
	
Vagrant.configure("2") do |config|
	#设置虚拟机的box
	config.vm.box = "centos7-base"
	
	#设置虚拟机的主机名
	config.vm.hostname = "centos7-base-jdk"
	
	#Virtualbox相关配置
	config.vm.provider "virtualbox" do |v|
		#设置虚拟机的名称
		v.name = "centos7-base-jdk"
		
		#设置虚拟机的内存大小为2G
		v.memory = 2048 
		
		#设置虚拟机的CPU个数
		v.cpus = 2 
	end 
	
	#使用shell脚本进行软件安装和配置
	config.vm.provision "shell" do |s| 
		s.path = "jdk-install.sh"
		s.env = {JDK_FILE: "/vagrant/jdk-8u181-linux-x64.tar.gz", 
			JDK_INSTALL_DIR: "/usr/java",
			JDK_INSTALL_PATH: "/usr/java/jdk1.8.0_181"}
	end 	
end
```


vagrant up拉起虚拟机，自动安装jdk， 安装结果： 

![java-version][4]


三、打包centos7-base-jdk成box，并加入到box list
创建一个目录，然后打开命令行，打包虚拟机成box，–base的参数必须是安装的centos7的虚拟机名称，本次是centos7-base-jdk。
再将生成的box添加到box list中，后续就可以直接使用centos7-base-jdk8作为镜像创建虚拟机了。
``` shell 
$ vagrant package --output centos7-base-jdk8.box --base centos7-base-jdk
$ vagrant box add centos7-base-jdk8 centos7-base-jdk8.box
$ vagrant box list
```

  [1]: centos7-install-jdk-build-box/tar-jdk.png
  [2]: centos7-install-jdk-build-box/set-etc-profile.png
  [3]: centos7-install-jdk-build-box/java-version.png
  [4]: centos7-install-jdk-build-box/one-key-install.png



