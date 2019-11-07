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

#  二、maven-shade-plugin介绍  

maven-shade-plugin是一个maven打包插件，提供的功能比较丰富，使用也简单易懂。 

### 1、简单打包

简单打包只需要增加execution, 指定执行package的phase，为这个phase绑定global shade就ok
```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

### 2、打包可运行的jar  

增加configuration, 添加ManifestResourceTransformer的transformer，指定main入口的mainClass

```xml
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <transformers>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>org.demo.App</mainClass>
                        </transformer>
                    </transformers>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```


### 3、打包第三方jar包和排除第三方jar包 

configuration下增加artifactSet，includes添加需要增加的第三方maven依赖，excludes排除不需要打包进来的第三方依赖，
include和exclude的标签里格式是maven的坐标和类型等， 格式：groupId:artifactId[[:type]:classifier],type可以是jar、war、pom, 
这里需要注意的是scope是test、provided的话，是无法include进来的，compile(默认)、runtime会被包含进来。 
```xml
                <configuration>
                    <artifactSet>
                        <includes>
                            <include>*:*</include>
                        </includes>
                        <excludes>
                            <exclude>org.projectlombok:lombok</exclude>
                        </excludes>
                    </artifactSet>
                    ...
                </configuration>    
```
也可以过滤第三方包中，指定那些文件打包进来，那些不打包。 
configuration下增加filters, filter说明过滤的条件和信息,include和exclude注意是文件路径，不是包路径。 
配置的每个filter,如果只配置了include，其他都会被exclude
```xml
                <configuration>
                    <filters>
                        <filter>
                            <artifact>javax.servlet.jsp:jsp-api</artifact>
                            <includes>
                                <include>javax/servlet/jsp/**</include>
                            </includes>
                            <excludes>
                                <exclude>javax/servlet/jsp/tagext/**</exclude>
                            </excludes>
                        </filter>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/NOTICE.txt</exclude>
                            </excludes>
                        </filter>
                    </filters>
                    ... 
                </configuration>
```

### 4、第三方包的包路径重命名

可以通过relocations，将依赖的第三方包打包到jar包中，并且重命名类的包名，可以达到隔离第三方包、解决包冲突等功能。 
如果有不需要导入的，可以用exclude排除掉。
```xml
                <configuration>
                    <artifactSet>
                        <includes>
                            <include>org.javassist:javassist</include>
                        </includes>
                    </artifactSet>
                    <relocations>
                        <relocation>
                            <pattern>javassist</pattern>
                            <shadedPattern>org.demo.javassist</shadedPattern>
                            <!--<excludes>
                                 <exclude>javassist.util.HotSwapAgent</exclude>
                            </excludes>-->
                        </relocation>
                        
                    </relocations>
                    ...
                </configuration>
```

### 5、其他配置

```xml

<!--打包的jar文件名，如果配置shadedArtifactAttached=true,该jar包不包含依赖的第三方包-->
<finalName>${project.name}-${project.version}</finalName>
<!-- true: 默认fasle, 最小化打包，去掉未用到的第三方包中的类,危险-->
<minimizeJar>false</minimizeJar>
<!--true: 默认true, 如果第三方包已经被打到jar中，依赖的jar的<dependencies>将会从jar包中的maven pom中删除-->
<createDependencyReducedPom>false</createDependencyReducedPom>
<!--true: 默认fasle, 打包的时候生成source jar,打包进去的第三方包也会把第三方包的sourc也打到source jar中-->
<createSourcesJar>false</createSourcesJar>
<!--生成classifier，配置finalName后生成的jar包名格式是${project.name}-${project.version}-${classifier}.jar-->
<!--默认会生成一个Jar包和一个以 “-shaded”为结尾的uber-jar包，可以通过配置来指定finalName打出来的的后缀名-->
<shadedArtifactAttached>true</shadedArtifactAttached>
<shadedClassifierName>suffix</shadedClassifierName>

```


# 三、maven-shade-plugin实战


把官网的例子和网上一些文章的例子试验了一下。 
其实，主要用到的应该是基本打包功能，jar包命名，ManifestResourceTransformer，这些功能其他打包插件也可以做得到。 
relocation功能比较使用，引入第三方包有特殊需求的时候，比如隔离和避免版本冲突，可以用该功能解决。 


```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.demo</groupId>
    <artifactId>maven-shade-plugin-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>maven-shade-plugin-demo</name>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!--scope=test,不会打包到jar中 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.28</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.60</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
            <scope>runtime</scope>
        </dependency>
        <!--scope=provided,不会打包到jar中 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.24.1-GA</version>
        </dependency>
        <!--第三方包依赖的其他包也会一并打到jar中 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>4.1.6.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <!--打包的jar文件名，如果配置shadedArtifactAttached=true,该jar包不包含依赖的第三方包-->
                    <finalName>${project.name}-${project.version}</finalName>
                    <!-- true: 默认fasle, 最小化打包，去掉未用到的第三方包中的类,危险-->
                    <minimizeJar>false</minimizeJar>
                    <!--true: 默认true, 如果第三方包已经被打到jar中，依赖的jar的<dependencies>将会从jar包中的maven pom中删除-->
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                    <!--true: 默认fasle, 打包的时候生成source jar,打包进去的第三方包也会把第三方包的sourc也打到source jar中-->
                    <createSourcesJar>false</createSourcesJar>
                    <!--生成classifier，配置finalName后生成的jar包名格式是${project.name}-${project.version}-${classifier}.jar-->
                    <!--默认会生成一个Jar包和一个以 “-shaded”为结尾的uber-jar包，可以通过配置来指定finalName打出来的的后缀名-->
                    <shadedArtifactAttached>true</shadedArtifactAttached>
                    <shadedClassifierName>suffix</shadedClassifierName>
                    <filters>
                        <!--更精细的指定那些文件要include那些要exclude-->
                        <filter>
                            <artifact>javax.servlet.jsp:jsp-api</artifact>
                            <includes>
                                <include>javax/servlet/jsp/**</include>
                            </includes>
                            <excludes>
                                <exclude>javax/servlet/jsp/tagext/**</exclude>
                            </excludes>
                        </filter>
                        <filter>
                            <artifact>*:*</artifact>

                            <!--问题1：Invalid signature file digest for Manifest main attributes
                            原因：有些jar包生成时，会 使用jarsigner生成文件签名（完成性校验），分为两个文件存放在META-INF目录下：

                            a signature file, with a .SF extension；
                            a signature block file, with a .DSA, .RSA, or .EC extension；-->
                            <excludes>
                                <exclude>META-INF/NOTICE.txt</exclude>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                    <!--打包第三方包-->
                    <artifactSet>
                        <includes>
                            <include>*:*</include>
                        </includes>
                        <excludes>
                            <exclude>org.projectlombok:lombok</exclude>
                        </excludes>
                    </artifactSet>
                    <!--重新命名依赖的第三方包的类包名-->
                    <relocations>
                        <relocation>
                            <pattern>javassist</pattern>
                            <shadedPattern>org.demo.javassist</shadedPattern>
                        </relocation>
                    </relocations>
                    <transformers>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <!--启动类-->
                            <mainClass>org.demo.App</mainClass>
                            <!--MANIFEST.MF文件的明细描述-->
                            <manifestEntries>
                                <Premain-Class>org.demo.agent.Agent</Premain-Class>
                                <Agent-Class>org.demo.agent.Agent</Agent-Class>
                                <Can-Redefine-Classes>true</Can-Redefine-Classes>
                                <Can-Retransform-Classes>true</Can-Retransform-Classes>
                                <Can-Set-Native-Method-Prefix>true</Can-Set-Native-Method-Prefix>
                            </manifestEntries>
                        </transformer>
                        <!--spi实现，如果多个包下存在相同接口的实现，可以用ServicesResourceTransformer合并-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                        <!--多个包下存在相同的资源名，内容需要合并，可以通过AppendingTransformer进行合并-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                            <resource>META-INF/spring.handlers</resource>
                        </transformer>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                            <resource>META-INF/spring.schemas</resource>
                        </transformer>
                        <!--xml文件的合并不是简单的append,需要特殊处理，避免破坏结构-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.XmlAppendingTransformer">
                            <resource>META-INF/magic.xml</resource>
                            <!-- Add this to enable loading of DTDs
                            <ignoreDtd>false</ignoreDtd>
                            -->
                        </transformer>
                        <!--国际化多语言的文件合并也需要特殊处理-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ResourceBundleAppendingTransformer">
                            <!-- the base name of the resource bundle, a fully qualified class name -->
                            <basename>path/to/Messages</basename>
                        </transformer>
                        <!--不需要合并的资源文件的列表-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.DontIncludeResourceTransformer">
                            <resources>
                                <resource>.txt</resource>
                                <resource>READ.me</resource>
                            </resources>
                        </transformer>
                        <!--可以把README.txt从resource打包的时候，放到META-INF/README... 试验没成功！！！-->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.IncludeResourceTransformer">
                            <resource>META-INF/README</resource>
                            <file>ReadMe.file</file>
                        </transformer>
                    </transformers>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
