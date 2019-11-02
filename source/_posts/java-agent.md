---

title: Java Agent
date: 2019-11-02 17:27:46
tags: [Java Agent,Instrument]
categories: java

---

[TOC]

# 一、Java Agent是什么 ？ 

Java Agent是什么，换句话说在那些地方能看到它的身影呢？  

### 1、热部署； 

- JRebel 
  一个实现快速热部署、节省大量重启时间，提高开发效率的插件，java应用启动时，启动参数设置-javaagent:jrebel路径/jrebel.jar
- spring-loaded
  一个JRebel的开源实现，VM启动参数设置 -javaagent:springloaded路径/springloaded.jar  -noverify

### 2、APM 
APM缩写是  Application Performance Management & Monitoring，应用程序的性能服务管理和监控
APM基本是参考Google的Dapper(大规模分布式跟踪系统)的体系来做的，主要对分布式系统的前后端处理、服务端调用的性能消耗进行跟踪。 

- skywalking  
  开源的APM系统， Java端的数据收集探针使用agent方式， 启动参数也要设置-javaagent:skywalking的agent路径/skywalking-agent.jar
- pinpoint
  另外一款开源的APM系统，通用采用agent探针， 启动参数-javaagent:pinpoint的agent路径/pinpoint-bootstrap.jar

### 3、线上诊断工具  
- arthas 
  阿里的开源的Java诊断工具，采用命令行模式交互， java -jar arthas-boot.jar后，选择进程pid，attach到对应的进程，加载agent包
- Btrace 
  另外一款诊断工具,提供按断可靠的动态跟踪分析功能，./bin/btrace -cp <classpath> <pid> <btrace-script> , attach到对应的java进程 

# 二、一个Java Agent Demo实例

看起来功能非常强大，但是怎么Java Agent长啥样，实现一个agent有那些需要基本的套路，开发了后怎么用。 

实现一个agent主要有两个注意点，1、编写一个类，提供premain方法， 

### 1、
  