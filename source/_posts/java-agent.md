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

实现一个agent主要有两个注意点： 
- 1、编写一个类，提供premain方法；
- 2、编写META-INF/MANIFEST.MF文件，指定Premain-Class成1编写的类

### 1、Premain-Class类

Premain-Class类必须提供premain静态方法， premain方法有两种形式： 

- a、public static void premain(String args, Instrumentation inst)
- b、public static void premain(String args) 

如果同时提供以上两种，带Instrumentation参数的优先级更高，无Instrumentation的不会被调用。 

args是随-javaagent:agent路径/java-agent.jar=args传入的args字符串，与main方法不同的是args只是一个字符串，不是字符串数组。

```java
package org.demo.java.agent;

import java.lang.instrument.Instrumentation;

public class Agent {
    public static void premain(String args, Instrumentation inst){
        System.out.println("================================Java Agent premain instrument======================");
        System.out.println("Agent  args: " + args);
        System.out.println("isRetransformClassesSupported: " + inst.isRetransformClassesSupported());
        System.out.println("isRedefineClassesSupported: " + inst.isRedefineClassesSupported());
        System.out.println("isNativeMethodPrefixSupported: " + inst.isNativeMethodPrefixSupported());
        System.out.println("Agent's ClassLoader:  " + Agent.class.getClassLoader().getClass().getName());
    }

    public static void premain(String args){
        System.out.println("================================Java Agent premain======================");
        System.out.println("Agent  args: " + args);
    }
}
```
### 2、MAINIFEST.MF文件 

MANIFEST.MF文件用来描述jar包的信息，存放在jar包的META-INFO目录下。 
java agent的jar包，需要用到该文件，用来描述agent运行时，程序的入口。
Premain-Class就是用来指定入口类的配置项。 


```yaml
Manifest-Version: 1.0
Premain-Class: org.demo.java.agent.Agent

```
注意MANIFEST.MF最后有一个空行。 

MANIFEST.MF可以使用MAVEN插件，在package的时候，一起生成。
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>single</goal>
            </goals>
            <phase>package</phase>
            <configuration>
                <!--一起打包依赖的jar-->
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <!--生成MANIFEST.MF文件到jar包的META-INF-->
                <archive>
                    <manifestEntries>
                        <Premain-Class>org.demo.java.agent.Agent</Premain-Class>
                    </manifestEntries>
                </archive>
            </configuration>
        </execution>
    </executions>
</plugin>   
```

# 三、怎么用agent

使用agent，只需要在启动参数增加-javaagent:agent路径/agent.jar=args，就可以使用agent。 

创建main方法的类：
```java
package org.demo;

public class App 
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}

```
执行App： 
```sh
>java -javaagent:agent的路径/java-agent-jar-with-dependencies.jar=hello  org/demo/App
================================Java Agent premain instrument======================
Agent  args: hello
isRetransformClassesSupported: false
isRedefineClassesSupported: false
isNativeMethodPrefixSupported: false
getAllLoadedClasses:
Agent's ClassLoader:  sun.misc.Launcher$AppClassLoader
Hello World!
```

如果多次设置javaagent会怎样呢？  

```sh
>java -javaagent:agent的路径/java-agent-jar-with-dependencies.jar=hello -javaagent:agent的路径/java-agent-jar-with-dependencies.jar=world  org/demo
/App
================================Java Agent premain instrument======================
Agent  args: hello
isRetransformClassesSupported: false
isRedefineClassesSupported: false
isNativeMethodPrefixSupported: false
getAllLoadedClasses:
Agent's ClassLoader:  sun.misc.Launcher$AppClassLoader
================================Java Agent premain instrument======================
Agent  args: world
isRetransformClassesSupported: false
isRedefineClassesSupported: false
isNativeMethodPrefixSupported: false
getAllLoadedClasses:
Agent's ClassLoader:  sun.misc.Launcher$AppClassLoader
Hello World!
```
执行结果是设置多少次，就执行多少次。 

# 四、 Instrumentation 

从上面的例子好像agent没啥卵用，除了输出点信息之外。 
但是我们注意到了两个参数的premain方法，有一个java.lang.instrument.Instrumentation。

我们来看看Instrumentation是个什么东西。 

### 1、Instrumentation介紹

类注释上说明了Instrumentation的目的： 
主要是提供开发工具代码所需的服务，用来将字节码注入到具体的方法，实现数据收集等功能，
这类工具一般不更改原程序的行为，只是添加一些附加的功能。
比如监控代理、探测器、覆盖率分析器和事件记录器等。 

也就是主要用来开发工具的，当然改变程序的行为也是可以做到的，要看使用场景，是生产上的使用，还是开发测试的辅助工具。

java agent机制，提供了启动时一个加载java编写的插桩服务的入口，Instrumentation提供注入修改class对象字节码能力的钩子入口。
这个钩子是通过一下两个方法实现钩子注入： 
- a、void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
- b、void addTransformer(ClassFileTransformer transformer); 

注入的钩子会在某个类的字节码文件读取之后，类定义之前被调用, 我们只需要实现ClassFileTransformer的transform方法，
在该方法中实现修改字节码，就能达到注入字节码的目的： 
```java 
byte[] transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException  
```

### 2、Instrumentation示例

现在我们有个需求，要记录某些方法的执行时间，具体方法运行前确定。
如果用AOP的方式，每次运行需要更改下AOP配置，是不是可以用java agent实现呢？  
从上面我们了解到的agent机制和instrumentation，思路可以整理下：
具体的方法和类，可以通过agent的args传入，然后在transform的时候修改对应的类方法，
注入记录时间的字节码。 

修改premain方法如下：

```java
   public static void premain(String args, Instrumentation inst){
        System.out.println("================================Java Agent premain instrument======================");
        System.out.println("Agent  args: " + args);
        String[] classMethods = args.split(";");

        final Map<String, Set<String>> classMethodMap = new HashMap<String, Set<String> > ();
        for (String classMethodList: classMethods) {
            int indexOfClass =  classMethodList.indexOf(":");
            String className = classMethodList.substring(0, indexOfClass);
            String[] methods = classMethodList.substring(indexOfClass+1).split(":");
            Set<String> methodSet = new HashSet<>();
            for (String methodName: methods) {
                methodSet.add(methodName);
            }
            classMethodMap.put(className, methodSet);
        }

        inst.addTransformer(new ClassFileTransformer() {
            @Override
            public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                                    ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
                String packageClass = className.replaceAll("/", ".");
                if(classMethodMap.containsKey(packageClass)){
                    CtClass ctClass = null;
                    Set<String> methodSet = classMethodMap.get(packageClass);
                    try {
                        ctClass = ClassPool.getDefault().makeClass(new ByteArrayInputStream(classfileBuffer));
                        if(!ctClass.isInterface()){
                            CtBehavior[] declaredBehaviors = ctClass.getDeclaredBehaviors();
                            for (CtBehavior behavior: declaredBehaviors) {
                                if(methodSet.contains(behavior.getName())){
                                    System.out.println("Inject byte code class: "+ packageClass + " method: " + behavior.getName());

                                    behavior.addLocalVariable("start", CtClass.longType);
                                    behavior.insertBefore("start = System.currentTimeMillis();");
                                    behavior.insertAfter("System.out.println(\"Method cost by agent...method: "+
                                            behavior.getName() + " cost: \" + (System.currentTimeMillis() - start ));");
                                }
                            }
                        }
                        return ctClass.toBytecode();
                    }catch(Exception ex){
                        ex.printStackTrace();
                    }finally{
                        if(ctClass != null){
                            ctClass.detach();
                        }
                    }
                }
                return classfileBuffer;
            }
        });
    }
```

这里需要引入字节码修改的库，用的是javassist： 
```xml
    <dependency>
      <groupId>org.javassist</groupId>
      <artifactId>javassist</artifactId>
      <version>3.24.1-GA</version>
    </dependency>
```

测试的App中增加一个方法longOperation： 
```java
package org.demo;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class App {
    public static void main( String[] args ) throws ClassNotFoundException, InterruptedException {
        System.out.println( "Hello World!" );

        log.info("Start long operation...");
        longOperation();
        log.info("End of long operation...");

    }

    public static void longOperation() throws InterruptedException {
        Thread.sleep(1000);
    }
}

```

然后更改启动参数： 
```bash
>java -javaagent:agent的路径/java-agent-instrument-jar-with-dependencies.jar=org.demo.App:longOperation  org/demo/App
================================Java Agent premain instrument======================
Agent  args: org.demo.App:longOperation:longOperation2
Inject byte code class: org.demo.App method: longOperation
Hello World!
20:26:40.519 [main] INFO org.demo.App - Start long operation...
Method cost by agent...method: longOperation cost: 1006
20:26:41.533 [main] INFO org.demo.App - End of long operation...

```

ok, 一个简单的agent就实现完毕了。 



# 五、总结

总的来说，实现一个agent并使用该agent还是挺简单的，只需要完成以下几步就可以搞定：
- 1、实现一个提供premain方法的类 
- 2、在MANIFEST.MF文件中，指定premain方法所在的入口类 
- 3、在premain方法中，将ClassFileTransformer的实现类，通过Instrumentation的addTransformer方法，设置类文件加载的回调钩子
- 4、打包agent成jar包
- 5、在目标应用启动时，指定-javaagent:agent的jar包路径=args

总的看来，机制简单，所以重点的内容还是在字节码修改，以便实现所需要的功能。 