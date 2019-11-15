---

title: 【译】javassist使用指南二(ClassPool)
date: 2019-11-14 12:23:03
tags: [javassist,bytecode]
categories: javassist

---

## 三、ClassPool 

ClassPool对象是CtClass对象的容器。 
一旦一个CtClass对象被创建，将会被记录到ClassPool对象中。 
这是因为编译器编译CtClass所包含的源码（通过API修改增加的源码）时，编译器需要访问该CtClass对象。 

比如，假设有一个新方法getter是一个被加入到一个Point类对应的CtClass对象。 
然后，程序尝试编译一段包含调用Point类的getter()方法源代码，这段源代码编译后将作为Line类的一个方法的方法体。 
如果Point对应的CtClass对象丢失了，编译器将无法编译该调用getter()方法的方法。
因为原来的Point类并不包含getter()方法。 
因此，为了正确编译类似的方法调用代码，ClassPool必须在整个程序执行过程中保存所有CtClass对象。

### 1、避免内存溢出 

ClassPool的这种方式在CtClass对象数量非常多的时候，可能会导致巨大的内存消耗（这种情况比较少发生，因为Javassist尝试各种方法去减少内存消耗）。
为了避免这种问题发生，你可以将一些不需要的CtClass对象从ClassPool移除掉。 
如果你调用CtClass对象的detach()方法，那么该CtClass将从ClassPool中移除。比如： 
```java
CtClass cc = ...; 
cc.writeFile(); 
cc.detach(); 
```

调用CtClass对象的detach()方法后，其他方法将不能被调用。 
但是，你能够通过ClassPool的get()方法， 重新创建一个代表对应类的CtClass对象。 
如果调用ClassPool的get()方法， ClassPool将重新读取一个类文件，并且重新创建一个CtClass对象，并通过get()方法返回。 

另一个方法（避免内存溢出）是用一个新的ClassPool对象替换老的对象，并将老的丢弃。 
如果一个老的ClassPool对象被垃圾收集，保存在ClassPool中的CtClass对象也将被垃圾收集掉。 
创建一个新的ClassPool对象，执行以下代码片段即可： 
```java
ClassPool cp = new ClassPool(true); 
//if needed, append an extra search path by appendClassPath() 
```
以上创建的ClassPool对象和ClassPool.getDefault()返回的默认ClassPool对象效果是一样的。 
ClassPool.getDefault()是一个便捷的单例工厂方法。
它创建的ClassPool对象与上述创建的ClassPool对象的方式一样，只是它保证创建的是一个ClassPool单例，以便重复使用。 
getDefault()方法返回的ClassPool对象没有什么特别之处。getDefault()方法只是一个便捷的方法而已。 

new ClassPool(true)是一个便捷的构造方法，用来创建一个ClassPool对象，并且将系统类搜索路径加入到其中。 
调用该构造方法等同于以下代码： 
```java
ClassPool cp = new ClassPool(); 
cp.appendSystemPath(); //or append another path by appendClassPath() 
```

### 2、级联的ClassPool

如果程序运行在一个web服务器上，可能需要创建多个ClassPool对象；每个ClassLoader需要创建对应的CLassPool对象。 
程序中应该用构造方法创建ClassPool对象，而不是直接调用getDefault()获得对象。

多ClassPool对象可以级联，与`java.lang.ClassLoader`类似。比如： 
```java
ClassPool parent = ClassPool.getDefault(); 
ClassPool child = new ClassPool(parent); 
child.insertClassPath("./classes"); 
```
如果调用child.get()方法，该子ClassPool会首先委托给父ClassPool。 
如果父ClassPool无法加载对应的类文件，那么子ClassPool会尝试从./classes目录下加载该类文件。 

如果child.childFirstLookup设置成true，子ClassPool会先尝试查找该类文件，未找到才会委托给父ClassPool. 比如：
```java
ClassPool parent = ClassPool.getDefault(); 
ClassPool child = new ClassPool(parent); 
child.appendSystemPath(); //the same class path as the default one. 
child.childFirstLookup = true; //changes the behavior of the child.
```

### 3、通过更改类名定义新类 

可以通过拷贝已经存在的类来定义一个新的类。如以下代码所示：
```java
ClassPool pool = ClassPool.getDefault(); 
CtClass cc = pool.get("Point"); 
cc.setName("Pair");
```

以上代码先获取类Point的CtClass对象。然后调用setName()方法，设置新的名字Pair。
以上调用后，CtClass对象中，类定义中涉及到的类名，将会从Point变成Pair。类定义的其他部分不会改变。
 
需要注意的是CtClass的setName()方法，更改了ClassPool中的记录。 从实现的角度看， ClassPool对象是一个保存CtClass对象的哈希表。 
setName()方法只是更改了哈希表中key对应CtClass对象的关系。Key从原来的类名变成了新的类名。 

因此，调用ClassPool对象的get("Point")方法，返回的将不再是cc引用的CtClass对象。
ClassPool对象将重新读取Point.class类文件，并重新为Point类创建一个新的CtClass对象。 
之所以这样，是因为CtClass对象关联的名字Point已经不存在于ClassPool中。 如下所示： 
```java
ClassPool cp = ClassPool.getDefault();
CtClass cc = cp.get("Point");
CtClass cc1 = cp.get("Point");  // cc1 is identical to cc.
cc.setName("Pair");
CtClass cc2 = cp.get("Pair");  // cc2 is identical to cc.
CtClass cc3 = cp.get("Point"); // cc3 is not identical to cc.
assertEquals(cc, cc1);  
assertEquals(cc1, cc2);
assertNotEquals(cc2, cc3);
```
cc1和cc2引用的是相同的CtClass实例，与cc一样，cc3是另外一个实例对象。 
注意：cc.setName("Pair")执行后，CtClass对象，cc和cc1引用的都都变成Pair类。 

ClassPool对象是用来维护类和CtClass对象之间的一对一关系。
Javassist不允许两个CtClass对象对应相同的类，除非两个CtClass对象分别由不同的ClassPool对象创建。 
这是保证转换一致性的重要特性。

创建一份默认ClassPool（从ClassPool.getDefault()返回的）实例对象的拷贝，执行以下代码片段（前面已经出现过）：
```java
ClassPool cp = new ClassPoo(true); 
``` 
如果你有两个ClassPool对象，你可以从这两个ClassPool对象获取到不一样的CtClass对象对应到相同的类文件。 
你可以分别修改这两个CtClass对象，并生成两个不同版本的Class类。 

### 4、通过重命名一个冻结类定义新的类

一旦一个CtClass对象通过writeFile()或toBytecode()方法转换成类文件，Javassist拒绝后续的对CtClass对象的修改。 
因此，Point类的CtClass对象转换成类文件后，因为在Point的CtClass上调用setName()会被拒绝，你不能通过执行setName()拷贝Point重新定义Pair类。
以下代码是无法运行的： 
```java
ClassPool pool = ClassPool.getDefault(); 
CtClass cc = pool.get("Point"); 
cc.writeFile(); 
cc.setName("Pair"); //wrong since writeFile() has bean called. 
```
绕开这个限制，你可以调用ClassPool的getAndRename()方法。 比如： 
```java
ClassPool pool = ClassPool.getDefault(); 
CtClass cc = pool.get("Point"); 
cc.writeFile(); 
CtClass cc2 = pool.getAndRename("Point", "Pair"); 
```
如果调用getAndRename()，ClassPool首先先读取Point.class，创建一个新的CtClass对象对应Point类。 
但是，在保存到ClassPool的哈希表之前，它将CtClass的名字从Point设置成Pair。 
因此，getAndRename()能够在Point的CtClass对象调用writeFile()或toBytecode()方法后被执行。 


