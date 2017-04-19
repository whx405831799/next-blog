title: 技术问题整理（持续更新）
date: 2016/12/10 19:06:06
categories:
- JAVA服务端
tags:
- 技术问题
---

**1.String、StringBuffer与StringBuilder之间区别**
- 执行速度上：StringBuilder >  StringBuffer  >  String（String 每次改变都会创建一个新的对象）。
- 线程安全上：String和StringBuffer是线程安全的，StringBuilder是非线程安全的。

**2.JVM加载class文件的原理机制**
- Class Loader加载class(Class Loader 只管加载，只要符合文件结构就加载，至于说能不能运行，则不是它负责的，那是由Execution Engine 负责的)。
- Execution Engine 执行引擎,执行引擎也叫做解释器(Interpreter) ，负责解释命令，提交操作系统执行。
- Native Interface 本地接口，本地接口的作用是融合不同的编程语言为Java 所用，现在用的越来越少了。
- Runtime data area 运行数据区，运行数据区是整个JVM 的重点。我们所有写的程序都被加载到这里，之后才开始运行。
- 整个JVM 框架由加载器加载文件，然后执行器在内存中处理数据，需要与异构系统交互是可以通过本地接口进行。
- 当执行 java ***.class 的时候， java.exe 会帮助我们找到 JRE ，接着找到位于 JRE 内部的 jvm.dll ，这才是真正的 Java 虚拟机器 , 最后加载动态库，激活 Java 虚拟机器。虚拟机器激活以后，会先做一些初始化的动作，比如说读取系统参数等。一旦初始化动作完成之后，就会产生第一个类加载器―― Bootstrap Loader ， Bootstrap Loader 是由 C++ 所撰写而成，这个 Bootstrap Loader 所做的初始工作中，除了一些基本的初始化动作之外，最重要的就是加载 Launcher.java 之中的 ExtClassLoader ，并设定其 Parent 为 null ，代表其父加载器为 BootstrapLoader 。然后 Bootstrap Loader 再要求加载 Launcher.java 之中的 AppClassLoader ，并设定其 Parent 为之前产生的 ExtClassLoader 实体。这两个加载器都是以静态类的形式存在的。这里要请大家注意的是， Launcher$ExtClassLoader.class 与 Launcher$AppClassLoader.class 都是由 Bootstrap Loader 所加载，所以 Parent 和由哪个类加载器加载没有关系。 
下面的图形可以表示三者之间的关系： 
BootstrapLoader <---(Extends)----AppClassLoader <---(Extends)----ExtClassLoader 

**3.char 型变量中能不能存贮一个中文汉字，为什么？**
- char类型可以存储一个中文汉字，因为Java中使用的编码是Unicode（不选择任何特定的编码，直接使用字符在字符集中的编号，这是统一的唯一方法），一个char类型占2个字节（16比特），所以放一个中文是没问题的。

**4.实现多态的方式：抽象类和接口的区别**
- 继承：抽象类只能继承一个，接口可以多继承。
- 语法：抽象类与普通类的区别是抽象类可以定义抽象方法，不用实现，供子类继承。私有方法是可以定义的，因为比如使用设计模式中的模板方法模式，就有必要定义private的方法。而接口，变量只能定义public final的常量，而且方法必须是public abstract的，缺省值也是这样。
- 意义上：抽象类，是对事物的抽象，对类抽象；而接口是对行为的抽象。一个经典的例子：门和警报。门一定有open和close的能力，但是不一定有警报的能力。所以如果把三个都放到门的抽象类里并不合适，因为不是所有门都会有警报，且把三个都放在接口里也不合适，因为有些能发警报的事务并不具有open和close的能力。结论就是抽象类里定义open和close,接口定义警报。