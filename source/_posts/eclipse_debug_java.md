---
title: 工具使用-Eclipse调试Java常用功能介绍
tags: eclipse调试
categories: 开发工具
---

## **调试常用功能介绍**

日常java开发中实际写代码的时间并不会很多，花在整理思路，代码重构，调试bug会占用很大部分时间。
熟练掌握调试工具，了解调试相关技巧，是java开发中减少调试时间，提高效率不可或缺的一部分。
使用System.out.println作为调试主要工具的方法，是java初学者对着书本例子敲代码的做法。

#### **一、调试相关的常用窗口**
**调试中最常用的窗口**是:

|窗口|说明|
|----------|----------|
|Debug窗口|主要显示当前线程方法调用栈， 以及代码行数(有调试信息的代码)|
|断点Breakpoints窗口|                  => 断点列表窗口，可以方便增加断点，设置断点条件，删除断点等|
|     变量Variables窗口   |                   => 显示当前方法的本地变量，非static方法，包含this应用，可以修改变量值|
|     代码编辑窗口         |                    => 这个不用多说了  |
|     输出Console窗口      |                 => 日志等输出内容，调试时，可以将关注的组件级别设置低一点，以便获得跟多输出信息|

另外**辅助的窗口**有:

|窗口|说明|
|----------|--------------------|
|     表达式expression窗口      |          =>  写上自己需要观察的数据的表达式，或者修改变量值|
|     Display窗口              |                 =>  可以在display中执行代码块，输出内容等|
|     大纲Outline窗口           |             =>  查看当前类的方法，变量等|
|     类型层级Type hierarchy窗口 |      => 查看当前所在类的继承层次，包括实现接口，类继承层次|
|     方法调用关系Call hierarchy窗口 | => 查看当前方法被哪些方法调用，调用方法在哪些类中、第几行，可以直接打开对应的方法|
|     搜索结果Search窗口             |      => 结合快捷键可以查看变量、方法等在工作空间、项目、工作集中被引用或定义的代码位置|
**窗口全览**：

![all_window_over_view][1]

#### **二、断点类型及断点窗口**
**在调试中可以设置的断点类型有五种**： 行断点(line breakpoints)，方法断点(method breakpoints )，观察断点(watch breakpoints-成员变量访问变更)，异常断点(exception breakpoints), 类加载断点(class load breakpoints)。每种断点的设置有些许不一样，可以在断点上右键->Breakpoint properties进行设置，但一般在断点窗口有快速设置的界面，Breakpoint properties中多了filter, 其实比较鸡肋，用处不大。

**断点相关的快捷键**：

|快捷键|说明|
|-----|--------------------|
|ctrl+shift+b|       在光标处大断点/取消断点  |
|ctrl+alt+b|          忽略所有断点|
|Alt+shift+q,   b|  激活断点窗口|


**1、<font color="red">行断点</font>**：  在方法中的某一行上打断点<font color="red">1.1、1.4</font>。行断点可以设置挂起线程/VM的条件<font color="red">1.3</font>，访问次数<font color="red">1.2</font>。
     <font color="red">1.3</font>中的条件是，spring在注册Bean定义(registerBeanDefinition)时，如果是org.springframework.demo.MyBean，就挂起线程，可以开始单步调试了。
     对于命中次数(hit count)<font color="red">1.2</font>的使用，一般是在循环中，第N个对象的处理有问题，设置hit count = N， 重调试时，可以方便到达需要调试的循环次数时，停下来调试。

**2、<font color="red">方法断点</font>**：在方法上打断点<font color="red">2.1、2.2</font>。方法断点的好处是可以从方法方法进入或者退出时<font color="red">2.3</font>，停下来调试，类似行断点，而且只有行断点和方法断点有条件和访问次数的设置功能。
     但是方法断点还有另外一个好处，如果代码编译时，指定不携带调试信息，行断点是不起作用的，只能打方法断点。
     有兴趣的可以通过A1将Add line number... 前的勾去掉， 调试下看看。

![line_and_method_breakpoints][2]

**3、<font color="red">观察断点</font>**： 在成员变量上打的断点<font color="red">3.1、3.3</font>。只有对象成员变量有效果，静态成员变量不起作用。
     可以设置变量被访问或者设置的时候挂起线程/VM <font color="red">3.2</font>，也就是类似<font color="red">3.4</font>的所有对成员变量的访问或者设置的方法都会被监控到
**4、<font color="red">异常断点</font>**： 异常断点可以通过<font color="red">4.6</font>添加，或者点击日志信息中输出的异常类信息添加。
     异常断点<font color="red">4.1</font>，系统发生异常时，在被捕获异常的抛出位置处或者程序未捕获的异常抛出处<font color="red">4.2、4.4</font>, 挂起线程/VM， 也可以指定是否包括异常的子类也被检测<font color="red">4.3、4.5</font>。
     另外除了以上正常设置的异常挂起，从java->debug中可以设置挂起执行，主要有下面两个： 1、是否在发生全局未捕获时挂起(suspend execution on uncaught exceptions)，调试时，老是有异常挂起影响调试，但是没有设置异常断点的情况，就可以勾选掉这个选项；2、是否在编译错误时挂起，一般在边调试边改代码时会发生这种情况；

另外要提一个的是有main方法启动的应用，可以在调试配置中勾选stop in main <font color="red">A3</font>, 程序进入时，会挂起线程，等待调试。

![watch_and_exception_breakpoints][3]

**5、类加载断点**: 在类名上打的断点<font color="red">5.1</font>。接口上是打不了类加载断点的，但是抽象类是可以的，只是在调试的时候，断点不会明显进入classloader中，单步进入知会进入到子类的构造方法中，非抽象类在挂起线程后单步进入就会到classloader中（如果没有filter过滤掉的话）<font color="red">5.3</font>。类加载断点不管是打在抽象或者非抽象类上，都会在类第一次加载或者第一个子类第一次被加载时，挂起线程/VM <font color="red">5.2</font>。

![class_load_breakpoints][4]

#### **三、Debug窗口、代码窗口，快捷键**

打完断点进入调试阶段，调试过程中最主要的就是调试窗口和代码窗口， 调试窗口和代码窗口的右键菜单、使用的快捷键以及达到的效果都值得深入了解；

**Debug窗口右键菜单及快捷键**：如<font color="red">A6</font>所示。
**代码窗口右键菜单及快捷键**：如<font color="red">A7</font>所示。

下面将按照功能，结合<font color="red">A6、A7</font>一起说明：

**1、<font color="red">调试执行</font>**：

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">6.4</font>| Step Info |          F5  |        ->  单步进入(如果有方法调用，将进入调用方法中进行调试); |
|<font color="red">6.4</font>| Step Over|          F6  |        ->  单步跳过(不进入行的任何方法调用中，直接执行完当前代码行，并跳到下一行); |
|<font color="red">6.4</font>| Step Return |      F7   |       ->  单步返回(执行完当前方法，并从调用栈中弹出当前方法，返回当前方法被调用处)； |
|<font color="red">6.5</font>| Resume|             F8   |       ->  恢复正常执行(直到遇到下一个断点); |
|<font color="red">7.4</font>| Run to Line |      Ctrl+R  |  ->  执行到当前行(将忽略中间所有断点，执行到当前光标所在行);   |
|<font color="red">6.3</font>| Drop To Frame |   无  |      ->  回退到指定方法开始处执行，这个功能<font color="red">相当赞</font>。<br/>在方法调用栈上的某个方法右键，选择Drop To Frame就可以从该方法的开始处执行<br/> 比如重新执行本方法，可以在本方法上用Drop To Frame，将从本方法的第一行重新执行。<br/>当然对于有副作用的方法，比如数据库操作，更改传入参数的对象内容等操作可能重新执行就再是你想要的内容了。|
|<font color="red">6.1+6.2</font>| Copy Stack |        无  |      ->  拷贝当前线程栈信息  |

如果在调试时，需要排除一些类、包等不需要进入调试，可以使用Edit Step Filters设置。
<font color="red">A6</font>的properties： Java进程启动相关信息，包括控制台启动参数，环境参数等。如果参数启动参数有问题可以先看看这里的实际启动参数是不是有误。另外还可以查看虚拟机支持调试的相关选项。

![debug_and_code_window][5]

**2、<font color="red">数据查看</font>**

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">7.4</font>|   Inspect    |              ctrl+shift+i   |       ->  察看选择的变量、表达式的值或执行结果, 再次按ctrl+shift+i可以将当前表达式或值添加到Expressions窗口中查看; |
|<font color="red">7.4</font>|   Display      |            ctrl+shift+d    |     ->  显示选择的变量、表达式的值或执行结果, 再次按ctrl+shift+d可以将当前表达式或值添加到Display窗口中显示; |
|<font color="red">7.4</font>|   Execute     |            ctrl+u        |          ->  执行选择表达式;| 
|<font color="red">7.4</font>|   Run to Line     |       ctrl+r         |         ->  执行到当前行(将忽略中间所有断点，执行到当前光标所在行);  |
|<font color="red">7.3</font>|   All Instances   |       ctrl+shift+n   |     ->  查看选择的类的所有对象，这个功能超赞；|
|<font color="red">7.3</font>|   Instance Count   |       无             |      ->  查看选择的类的所有对象个数；|
|<font color="red">7.4</font>|   Watch         |              无         |          ->   添加当前变量、表达式到Expressions窗口中；|

#### **四、调试时窗口**

**1、<font color="red">Display窗口A8</font>**
     在Display窗口中，可以执行代码之外的任何合理的语句，也可以使用Inspect， Display, Execute这些方式查看或者执行语句——<font color="red">8.1</font>。
比如， **a**、输入com.alibaba.fastjson.JSON.toJSONString(registry, true); 并选择这一行代码，ctrl+shift+d，可以执行这行代码并在Display窗口中输出结果——<font color="red">8.2</font>。
**b**、当然也可以改变当前代码的变量值，比如输入beanName = "myBeanX"; 选择这行代码，用Inspect， Display, Execute这几种方式都可以执行这行代码。
**c**、也可以输入System.out.println(com.alibaba.fastjson.JSON.toJSONString(registry, true)); 执行就会在console窗口输出结果。

**2、Variables窗口A9**
     在Variable窗口中可以看到当前方法的本地变量值<font color="red">8.3</font>，非静态方法会有this表示对象自己，也可以查看本地变量相应的属性<font color="red">8.4</font>， <font color="red">8.5</font>的功能可以让集合类型对象中的数据显示容易看点。
这个窗口除了察看变量的功能之外，还可以直接在Value列修改变量值或对象属性。

**3、Expression窗口A10**
    Expression窗口的功能主要是用来察看变量之外的表达式的结果，执行固定表达式等功能。比如察看变量definitionHolder中的属性beanDefinitionHolder, 直接将definitionHolder.getBeanDefinition()加入到Expressions窗口<font color="red">8.6， 8.7</font>。

![display_variables_expression_window][6]

#### **五、代码查看辅助窗口**
**1、代码窗口**：

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">11.1~11.5</font>  |quick type hierarchy |     ctrl+t  |   查看当前类、接口的继承层次， 默认进入时，显示继承/实现当前类/方法的子类，子接口11.1；再次ctrl+t，将显示当前类、接口继承/实现的超类/接口11.2；调试时，经常用该功能，在接口或抽象类的方法调用处11.3，ctrl+t察看实现类11.4，直接导航到对应的实现方法中11.5。|
||quick outline  |                ctrl+o|    查看当前类的大纲，包括方法，属性等内容； 用处不大； |
||open declarations     |      F3   |      查看变量、属性、方法定义的地方| 

![code_window_hotkeys][7]

**2、Call Hierarchy窗口**:

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">12.1~12.2</font> |open call hierarchy|        ctrl+alt+h |  查看方法被调用层次， 可以看当前方法被调用的地方<font color="red">12.1</font>，或者当前方法调用了其他类的方法<font color="red">12.2</font> |  

![call_hierarchy][8]

**3、Type Hierarchy窗口**：

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">13.1~13.4 </font> | open type hierarchy|  F4  | 查看继承层次， 可以查看类的继承层次，包括子类父类<font color="red">13.1</font>， 或者类实现的接口继承层次<font color="red">13.2</font>，还会根据选择的类/接口，在右边显示该类的大纲<font color="red">13.3、 13.4</font>； <font color="red">13.3</font>可以选择是否显示父类/父接口的属性、方法等  |

![type_hierarchy][9]

**4、Search 窗口**：

|标记|功能|快捷键|描述|
|----|----------|------|---------------------|
|<font color="red">14.1  </font> |declarations  |                ctrl+g      |      相同的方法签名在工作空间中及第三方jar包中被定义的位置<font color="red">14.1 |
|<font color="red">14.2  </font> |references  |                  ctrl+shif+g   | 当前选中的变量、属性、方法在工作空间中及第三方jar包中被引用的位置<font color="red">14.2</font> | 
|<font color="red">14.3 </font> |   |                                ctrl+shift+u |    查看变量、属性 、方法在当前类中出现的地方<font color="red">14.3</font> |
|<font color="red">14.4</font> |  implements    |                 |                   查看实现当前接口的类<font color="red">14.8</font> |
|<font color="red">14.4~14.7</font> |  显示方式    |               |                可以选择不同的显示方式，具体看图                |

![search_window][10]


  [1]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/all_window_over_view.png?raw=true
  [2]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/line_and_method_breakpoints.png?raw=true
  [3]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/watch_and_exception_breakpoints.png?raw=true
  [4]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/class_load_breakpoints.png?raw=true
  [5]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/debug_and_code_window.png?raw=true
  [6]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/display_variables_expression_window.png?raw=true
  [7]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/code_window_hotkeys.png?raw=true
  [8]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/call_hierarchy.png?raw=true
  [9]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/type_hierarchy.png?raw=true
  [10]: https://github.com/run-zheng/tool_use/blob/master/images/eclipse_debug/search_window.png?raw=true
