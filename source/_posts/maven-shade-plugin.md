---

title: maven-shade-plugin介绍
date: 2019-11-06 19:19:39
tags: [maven-shade-plugin] 
categories: maven

---


[TOC]

# 一、 缘起

编写java agent插件的时候，用到javassist修改字节码，插件用来记录调用链的，需要在方法的前后插入代码。 
突发奇想，用来看看javassist是怎么调用的，结果达不到预期效果，因为java agent中，javassist的代码已经加载过了，
没插入记录调用链的代码，刚好看到guava中有介绍用maven-shade-plugin将guava repackage重命名包名，因此记录下。 

# 二、maven-shade-plugin介绍  

maven-shade-plugin是一个maven打包插件，提供的功能比较丰富，使用也简单易懂。 

1、简单打包

