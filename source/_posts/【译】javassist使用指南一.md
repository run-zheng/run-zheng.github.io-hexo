---

title: 【译】javassist使用指南一（前言、读写字节码）
date: 2019-11-12 12:37:08
tags: [javassist,bytecode]
categories: javassist

---

# 一、前言 

Javassist(Java Programming Assistant)让Java字节码操纵变得简单。
它是一个在Java开发中编辑字节码的类库；它能够让java程序在运行时定义新的类，也可以在jvm加载类的时候修改类文件。
与其他字节码编辑类库不同的是，Javassist提供了两个层次的API：源码级别和字节码级别。
如果用户使用源码级别的API，可以在不了解Java字节码规范的情况下编辑类文件，整个源码级别的API按照Java语言风格进行设计。
你甚至可以将源文本插入到指定的字节码中，Javassist会将源文本进行即时编译。
另外一方面，字节码级别的API允许用户像使用其他字节码编辑类库一样编辑类文件。

> [参考官网介绍](http://www.javassist.org/)

# 二、读写字节码

Javassist是一个处理字节码的类库。Java的字节码存储在二进制的类文件中。每个类文件包含一个类或接口。 
Javassist.CtClass是类文件的抽象表示。一个CtClass(compile-time class编译时类)对象是处理类文件的一个句柄。
下列是个简单的示例： 

```java
ClassPool cp = ClassPool.getDefault();
CtClass cc = cp.get("test.Rectangle");
cc.setSuperclass(cp.get("test.Point"));
cc.writeFile();
```

这段代码首先获取一个ClassPool对象，用来在Javassist中控制字节码的修改。 
ClassPool对象是CtClass对象的容器，CtClass对象是类文件的抽象表示。 
ClassPool对象根据需要读取类文件创建CtClass对象，并且记录创建的CtClass对象以便后续访问。
要修改类的定义，用户首先需要从ClassPool对象得到表示该类CtClass对象的引用，
ClassPool的get()方法就是用来干着活的。
上述例子中，表示类test.Rectangle的CtClass对象是从ClassPool对象中获取的，并且将引用赋值给了cc变量。 
从ClassPool的getDefault()方法得到的ClassPool对象，搜索的类路径默认是系统类路径。 

从实现的角度讲，ClassPool是一个存储CtClass对象的哈希表，类名作为键。
ClassPool的get()方法搜索该哈希表，查找给定key关联的CtClass对象。 
如果没有找到一个CtClass对象，get()方法会读取一个类文件，创建一个新的CtClass对象，并存储到该哈希表中，然后将该对象作为get()方法的返回值返回。 

从ClassPool对象中获取的CtClass对象，是可以修改的([修改CtClass的详细介绍](http://www.javassist.org/tutorial/tutorial2.html#intro)会在后续讲解)。
上述的例子中，test.Rectangle的CtClass对象已经被修改了，父类由Object更改成test.Point。
当最终调用CtClass对象的writeFile()方法时，这个改变将会在原来的class文件中反映出来。 

```bash

$ javap -c target/classes/test/Rectangle.class   #原字节码反编译 
Compiled from "Rectangle.java"
public class test.Rectangle {
  public test.Rectangle();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
}

$ javap -c test/Rectangle.class   #writeFile出来的字节码反编译
Compiled from "Rectangle.java"
public class test.Rectangle extends test.Point {
  public test.Rectangle();
    Code:
       0: aload_0
       1: invokespecial #18                 // Method test/Point."<init>":()V
       4: return
}

```

writeFile()方法用来将CtClass对象转换成类文件并写入到本地磁盘。 
Javassist也提供了一个直接获取CtClass对象被修改过的字节码。 
获取字节码可以通过调用toBytecode()： 

```java
byte[] b = cc.toBytecode(); 
```

你也可以直接加载被修改过的CtClass对象的字节码： 

```java
Class clazz = cc.toClass(); 
```

toClass()方法要求有一个上下文ClassLoader，以便当前线程加载CtClass代表的类文件。
该方法将返回一个java.lang.Class对象代表被加载的类。
更多详细内容请[参考下文](http://www.javassist.org/tutorial/tutorial.html#toclass)

## 1、定义新类

要从无到有定义一个新类，必须调用ClassPool的makeClass()方法。 

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Point");
```

上述例子定义了一个类Point，没有成员信息。 
Point的成员方法可以通过CtNewMethod声明的工厂方法创建，然后通过CtClass的addMethod方法添加到Point类中。 

makeClass()不能创建一个接口。 
定义接口必须使用ClassPool的makeInterface()方法。 
接口中的成员方法可以通过CtNewMethod的absractMethod()方法创建。
> 注：接口中的方法是在字节码层面体现出来的是抽象方法

## 2、冻结类

如果一个CtClass对象已经调用writeFile()、toClass()或toByteCode()转换成一个类文件/类对象，Javassist会冻结该CtClass对象。 
不允许后续对该CtClass对象的进行修改。 
这是为了警告开发人员，他们修改的类文件已经被加载， JVM不允许修改已加载的类。 

一个被冻结的CtClass对象可以解冻，解冻后将允许修改该类的定义。比如： 

```java 
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Rectangle");
cc.writeFile();
cc.defrost();
CtClass pcc = pool.makeClass("Point");
cc.setSuperclass(pcc);
```

调用defrost()方法后， CtClass对象将再次可以修改。 

如果ClassPool.doPruning设置为true, 那么Javassist在冻结CtClass对象的时候，会修剪(精简)CtClass对象包含的数据结构。
为了减少内存的消耗，修剪过程会丢弃CtClass对象中不必要的属性（attribute_info结构）。
比如，Code_attribute结构(方法体)将被丢弃。
因此，一个CtClass对象被修剪后，除了方法名称，签名和注解，方法的字节码将无法访问。 
修剪过的CtClass对象无法被解冻。 
ClassPool的doPruning属性默认是false。

特定的CtClass对象要禁用修剪功能，必须在该对象上先调用stopPruning()方法(writeFile/toBytecode/toClass前)。

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Rectangle");
CtClass pcc = pool.makeClass("Point");
cc.stopPruning(true);
cc.setSuperclass(pcc);
pcc.writeFile();
cc.writeFile();
```

CtClass对象cc未被修剪。因此可以在调用writeFile()后进行解冻。 

```java
ClassPool pool = ClassPool.getDefault();
pool.doPruning = true; 

CtClass pcc = pool.makeClass("Point");
pcc.stopPruning(true); //super class must not proned too . 
pcc.writeFile();

CtClass cc = pool.makeClass("Rectangle");
cc.stopPruning(true);
cc.writeFile();  //convert to a class file

cc.defrost(); //cc is not pruned
cc.setSuperclass(pcc);
pcc.writeFile();
```

> 注： 在调试时，你可能想要临时停止修剪和冻结，并将修改后的字节码写会磁盘的类文件。 
使用debutWriteFile()方法可以方便的达到该目的。
该方法停止修剪，写类文件，然后解冻CtClass对象，重新打开修剪开关(如果原来已经打开)。

## 3、类搜索路径 

由静态方法ClassPool.getDefault()返回的默认ClassPool的类搜索路径和JVM的搜索路径一致。 
_ 如果程序运行在JBoss或Tomcat等web应用服务器上，ClassPool对象可能会找不到用户定义的类，
 因为这些web服务器使用多个类加载器（ClassLoader）来加载类，也包括系统类加载器（System ClassLoader）_。
 因此必须将其他类路径注册到ClassPool。假设pool引用的是ClassPool对象，如下： 

```java
pool.inertClassPath(new ClassClassPath(this.getClass()));
```

以上语句将加载this所属类对象的类路径注册到ClassPool对象。 
你可以使用任何类对象作为参数，替代this.getClass()。 
用于加载该类对象的类路径将被注册到pool中。 

你能够注册一个目录作为类搜索路径。
如下示例，以下代码会将目录`/usr/local/javalib`加入搜索路径：
 
```java
ClassPool pool = ClassPool.getDefault(); 
pool.insertClassPath("/usr/local/javalib"); 
```

用户可以加入到类路径中的，不仅仅是目录，URL也可以：
 
```java
ClassPool pool = ClassPool.getDefault();
ClassPath cp = new URLClassPath("www.javassist.org", "80", "/java/", "org.javassist."); 
pool.insertClassPath(cp); 
```

上述代码，将"http://www.javassist.org:80/java/"加入到类搜索路径中。 
这个URL仅在搜索属于`org.javassist`包下的类时，会被用到。 
比如，加载类`org.javassist.test.Main`, 该类文件将会从以下链接获取：

`http://www.javassist.org:80/java/org/javassist/test/Main.class` 

此外，你还可以通过ClassPool对象利用字节码数组创建一个CtClass对象。 
要做到这一点，必须使用ByteArrayClassPath。如下面例子： 

```java
ClassPool cp = ClassPool.getDefault();
String path = this.getClass().getResource("../../test/Rectangle.class").getPath();
File file = new File(path);
try(FileChannel channel = new FileInputStream(file).getChannel();){
    ByteBuffer bb = ByteBuffer.allocate((int) channel.size());
    channel.read(bb);
    byte[] bytes = bb.array();
    String name = "test.Rectangle";
    cp.insertClassPath(new ByteArrayClassPath(name, bytes));
    CtClass cc = cp.get(name);
    cc.writeFile();
}
```

上面例子中，得到的CtClass对象代表的是，由字节码数组bytes定义的类。 
调用ClassPool对象的get方法时，如果制定的name与设置的ByteArrayClassPath的name匹配，将会从该ByteArrayClassPath读取类文件。

如果你不清楚类全路径名，但可以得到类文件的输入流，可以通过ClassPoll的makeClass方法得到CtClass对象：
 
```java
ClassPool cp = ClassPool.getDefault();
String path = this.getClass().getResource("../../test/Rectangle.class").getPath();
File file = new File(path);
try(FileInputStream fis = new FileInputStream(file);){
    CtClass cc = cp.makeClass(fis);
    cc.writeFile();
}
```

makeClass()方法返回从输入流创建的CtClass对象。 
你可以使用makeClass()方法将类文件直接加载进ClassPool对象。 
这种方式在类搜索路径中包含大的jar文件时，可以提升性能。 
因为一个ClassPool对象是按需读取类文件的，这样可能会重复搜索整个jar包中的每个文件。 
makeClass()在这种情况下，可以用来优化这种情况下的类搜索。 
通过makeClass()创建的CtClass对象将被缓存在ClassPool对象中，后续将不会再读取该CtClass对象对应的类文件。

用户可以扩展类搜索路径。可以定义一个类实现ClassPath接口，并通过insertClassPath方法将该类的对象加入到ClassPool对象中。 
这种方式将允许非标准的资源加入到类搜索路径中。  





 
 

