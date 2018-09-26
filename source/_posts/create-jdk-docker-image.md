---

title: Dockerfile构建jdk镜像和tomcat镜像
date: 2018-09-26 23:30:53
tags: [docker,jdk镜像,tomcat镜像,docker镜像]
categories: docker


---

[TOC]

## 一、用vagrant up拉起一个基于docker的centos7虚拟机

``` ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.box = "centos7-base-docker"

  config.vm.hostname="centos7-docker-standalone"

  config.vm.synced_folder "./share", "/home/vagrant/share"

  config.vm.network "private_network", ip: "192.168.10.10"

  config.vm.provider "virtualbox" do |v|
	v.name = "centos7-docker-standalone"
	v.memory = 4096
	v.cpus = 2
  end
end
```

## 二、拉取一个centos7的镜像
``` sh
$ docker pull centos
$ docker run -it centos /bin/bash
  /]# cat /etc/redhat-release
```

![pull-centos7-images            ][1]

## 三、准备Dockerfile，构建centos7-jdk8镜像并验证
#### 1、准备Dockerfile
``` docker
FROM centos
MAINTAINER jacky zheng
ENV REFRESHED_AT 2018-09-25 23:45

ADD jdk-8u181-linux-x64.tar.gz /usr/java/

ENV JAVA_HOME /usr/java/jdk1.8.0_181
ENV CLASSPATH $JAVA_HOME/lib;$JAVA_HOME/jre/lib
ENV PATH $PATH:$JAVA_HOME/bin
```

![prepare-centos7-jdk8-dockerfile][2]

#### 2、利用docker build -t jacky/centos7-jdk8 . 构建镜像
``` sh
$ docker build -t jacky/centos7-jdk8 .
```

![build-centos7-jdk8-image       ][3]


#### 3、验证构建的镜像是否正确
``` sh
$ docker run -it jacky/centos7-jdk8 /bin/bash
  /]# java -version
  /]# javac -version
```

![run-a-centos7-jdk8-container   ][4]


## 四、利用构建的jdk镜像，构建tomcat镜像
#### 1、准备构建tomcat镜像的Dockerfile
``` docker
FROM jacky/centos7-jdk8
MAINTAINER jacky zheng 
ENV REFRESHED_AT 2018-09-26 22:45

ADD apache-tomcat-9.0.12.tar.gz /usr/tomcat/

ENV CATALINA_HOME /usr/tomcat/apache-tomcat-9.0.12
ENV CATALINA_BASE $CATALINA_HOME
ENV PATH $PATH:$CATALINA_HOME/lib:$CATALINA_HOME/bin

#暴露端口8080
EXPOSE 8080

#启动时运行tomcat
CMD ["/usr/tomcat/apache-tomcat-9.0.12/bin/catalina.sh", "run"]

```

#### 2、构建tomcat镜像，并验证
``` sh
$ ls -l
$ docker images
$ docker ps -a
$ docker build -t jacky/centos7-tomcat9 .
$ docker images
$ docker run -d -p 8081:8080 --name tomcat9-test jacky/centos7-tomcat9
```

![build-centos7-tomcat9-image    ][5]
![run-a-centos7-tomcat9-container][6]
![test-tomcat9                   ][7]


 [1]: create-jdk-docker-image/pull-centos7-images.png
 [2]: create-jdk-docker-image/prepare-centos7-jdk8-dockerfile.png
 [3]: create-jdk-docker-image/build-centos7-jdk8-image.png
 [4]: create-jdk-docker-image/run-a-centos7-jdk8-container.png
 [5]: create-jdk-docker-image/build-centos7-tomcat9-image.png
 [6]: create-jdk-docker-image/run-a-centos7-tomcat9-container.png
 [7]: create-jdk-docker-image/test-tomcat9.png










