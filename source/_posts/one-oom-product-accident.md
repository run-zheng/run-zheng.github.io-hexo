---

title: 记一次生产内存泄漏问题
date: 2018-03-26 23:00:40
tags: [OOM,virtual vm,MAT,pinpoint]
categories: 问题处理

---




**时间：**  2018年3月16号晚

**表现现象：** 客户访问非常慢到最后无法打开

**pinpoint请求：**  请求逐步变慢（[图1](#图1)），持续Full GC&CPU占用率高([图2](#图2))，出现OOM（[图3](#图3)）

**紧急措施：**  加大内存、重启服务器 
**重现问题：**  测试环境模拟并发访问（10线程&10000次/线程），开jmx端口&Virtual VM监控（[图4](#图4)），问题重现，启动两三分钟后开始出现GC，后续内存持续上涨，dump堆转储文件

**分析问题：** [MAT(Memory Analyzer tool)](
http://www.eclipse.org/mat/)分析，找泄露的代码


1. 从MAT的分析可以看出， ```javax.crypto.JceSecurity``` 的实例占用了最多的堆内存(Retained Heap | 深堆) （[图5](#图5)）

2. 从dominated tree可以看到，```javax.crypto.JceSecurity``` 的实例的retained heap占了73% （[图6](#图6)），主要是一个IdentityHashMap类型的属性verificationResults，放了很多```org.bouncycastle.jce.provider.BouncyCastleProvider``` 对象， 每个的retained heap占用182712字节 （[图7](#图7)） 

3. 另外从virtual vm看转储堆上的线程， 有很多BLOCKED的线程， 都卡在```javax.crypto.JceSecurity.getVerificationResult(JceSecurity.java:173)``` （[图8](#图8)），按现象看应该是在等待Full GC 

4. 从getVerificationResult可以看出， 只要新传入Provider都会放到verificationResults缓存起来， 
看调用链上（Rsa的decrypt [图11](#图11)-> Cipher.getInstance [图12](#图12) -> JceSecurity.getVerificationResult [图13](#图13)），Rsa的解密不应该每次重新```new org.bouncycastle.jce.provider.BouncyCastleProvider``` 对象。 

5. BouncyCastleProvider的问题（[图12](#图12)、[13](#图13)）：

 a. provider自己是一个java.util.Properties，将所有的预设的provider的key， value都作为property put到自己的hashtable里；

 b. 如果key, value都是string的话， 还将他们放到一个```java.util.LinkedHashMap.LinkedHashMap<String,String>()``` 的属性 legacyStrings里； 

 c. 在```java.security.Provider.getService(String, String)``` 的时候， 会把legacyStrings里所有的key,value解析成ServiceKey,Service对，放到另一个```java.util.LinkedHashMap.LinkedHashMap<ServiceKey,Service>()``` 的属性legacyMap里

 d. 因此，每次new BouncyCastleProvider 都会产生非常多的对象和引用（占用182712字节），且缓存在JceSecurity的verificationResults没法释放。  


**解决问题：** 按照BouncyCastleProvider 的注释，改代码，将BouncyCastleProvider实例对象作为Rsa的静态成员变量可以解决问题。 

**验证上线：** 更改后再验证（[图14](#图14)），内存使用正常，CPU正常

> **备       注：** 其实， 在重现问题，设置jmx端口，用jmeter测试的时候，已经开始在看代码， 因为最近加就只加了解密的代码，除了```Cipher.getInstance("RSA", new org.bouncycastle.jce.provider.BouncyCastleProvider());``` 这句，其他地方看不出来会出现内存泄露，于是，顺着看下去，就看到```javax.crypto.JceSecurity.getVerificationResult``` 里的缓存，回头看了下BouncyCastleProvider就确定怀疑正确。 然后紧急版本线上了再说。virtual vm和MAT的截图都是后续再分析时候截的。 

<span id="图1">图1</span>
![response_from_pinpoint][1]

<span id="图2">图2</span>
![full_gc_oom][2]

<span id="图3">图3</span>
![oom_of_pinpoint_trace][3]

<span id="图4">图4</span>
![monitor_by_virtualvm][4]

<span id="图5">图5</span>
![mat_analyze_summary][5]

<span id="图6">图6</span>
![mat_dominator_tree][6]

<span id="图7">图7</span>
![mat_list_objects_of_jcesecurity][7]

<span id="图8">图8</span>
![virtualvm_thread_blocked][8]

<span id="图9">图9</span>
![jcesecurity_getverifycationresult][9]

<span id="图10">图10</span>
![cipher_getinstance][10]

<span id="图11">图11</span>
![bad_code][11]

<span id="图12">图12</span>
![bouncycastleprovilder][12]

<span id="图13">图13</span>
![jcesecurity_three_map][13]

<span id="图14">图14</span>
![solve_the_problem][14]


  [1]: one-oom-product-accident/response_from_pinpoint.png
  [2]: one-oom-product-accident/full_gc_oom.png
  [3]: one-oom-product-accident/oom_of_pinpoint_trace.png
  [4]: one-oom-product-accident/monitor_by_virtualvm.png
  [5]: one-oom-product-accident/mat_analyze_summary.png
  [6]: one-oom-product-accident/mat_dominator_tree.jpg
  [7]: one-oom-product-accident/mat_list_objects_of_jcesecurity.png
  [8]: one-oom-product-accident/virtualvm_thread_blocked.jpg  
  [9]: one-oom-product-accident/jcesecurity_getverifycationresult.png
  [10]: one-oom-product-accident/cipher_getinstance.png
  [11]: one-oom-product-accident/bad_code.png
  [12]: one-oom-product-accident/bouncycastleprovilder.jpg
  [13]: one-oom-product-accident/jcesecurity_three_map.png
  [14]: one-oom-product-accident/solve_the_problem.png