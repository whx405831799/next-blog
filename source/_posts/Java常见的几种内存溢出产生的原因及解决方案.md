title: Java常见的几种内存溢出产生的原因及解决方案
date: 2017/04/11 12:21:35
categories:
- JVM
tags:
- 内存溢出
---

## 1.栈溢出-StackOverflowError
### 1.1 解释
- java方法执行的内存模型,每个方法执行都会创建一个栈帧，用于存储局部变量表，操作栈，动态链接，方法出口等信息，每一个方法被调用直至执行完成的过程。
- 栈溢出抛出java.lang.StackOverflowError异常，出现此种情况是因为方法运行的时候栈的深度超过了虚拟机容许的最大深度所致。一般情况下是程序错误所致的，比如写了一个死递归，就有可能造成此种情况。例如下面例子：

```
/**
 * 测试栈溢出
 */
public class TestStackOverflow {
    public void testRecursion(){
        testRecursion();
    }

    public static void main(String[] args) {
        TestStackOverflow testStackOverflow = new TestStackOverflow();
        testStackOverflow.testRecursion();
    }
}

Exception in thread "main" java.lang.StackOverflowError
	at jvm.TestStackOverflow.testRecursion(TestStackOverflow.java:8)
	at jvm.TestStackOverflow.testRecursion(TestStackOverflow.java:8)
	at jvm.TestStackOverflow.testRecursion(TestStackOverflow.java:8)
	at jvm.TestStackOverflow.testRecursion(TestStackOverflow.java:8)
```

### 1.2 解决
检查程序的错误。


## 2.堆溢出-OutOfMemoryError:java heap space
### 2.1 解释及处理方式
- 堆内存溢出的时候，虚拟机会抛出java.lang.OutOfMemoryError:java heap space,出现此种情况的时候，我们需要根据内存溢出的时候产生的dump文件来具体分析（需要增加-XX:+HeapDumpOnOutOfMemoryErrorjvm启动参数）。出现此种问题的时候有可能是内存泄露，也有可能是内存溢出了。要具体分析。
- 如果内存泄露，我们要找出泄露的对象是怎么被GC ROOT引用起来，然后通过引用链来具体分析泄露的原因。如果出现了内存溢出问题，这往往是程序本生需要的内存大于了我们给虚拟机配置的内存，这种情况下，我们可以采用调大-Xmx来解决这种问题。
- -Xms 初始堆大小，默认大小物理内存的64分之1，不超过1G；默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制。
- -Xmx 最大堆大小，默认大小为物理内存的4分之1，不超过1G；默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。
- -Xmn 	年轻代大小,sun推荐8分之3。
### 2.2 例子测试

```
/**
 * 测试outofmemory
 */
public class TestOutOfMemory {
    public static void main(String args[]){
        List<byte[]> buffer = new ArrayList<>();
        buffer.add(new byte[10*1024*1024]);
    }
}
```
我们在运行main方法时加上jvm参数：

```
-Xmn10M -Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError -Xloggc:D:\heap_trace.txt 
```

结果为：

```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid12708.hprof ...
Heap dump file created [1787890 bytes in 0.021 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at jvm.TestOutOfMemory.main(TestOutOfMemory.java:12)
```
GC信息为：

```
0.381: [GC [PSYoungGen: 1348K->600K(9216K)] 1348K->600K(19456K), 0.0320335 secs] [Times: user=0.00 sys=0.00, real=0.03 secs] 
0.413: [GC [PSYoungGen: 600K->568K(9216K)] 600K->568K(19456K), 0.0157791 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
0.429: [Full GC [PSYoungGen: 568K->0K(9216K)] [ParOldGen: 0K->524K(10240K)] 568K->524K(19456K) [PSPermGen: 2863K->2862K(21504K)], 0.1693650 secs] [Times: user=0.16 sys=0.00, real=0.17 secs] 
0.599: [GC [PSYoungGen: 0K->0K(9216K)] 524K->524K(19456K), 0.0651178 secs] [Times: user=0.06 sys=0.00, real=0.07 secs] 
0.664: [Full GC [PSYoungGen: 0K->0K(9216K)] [ParOldGen: 524K->512K(10240K)] 524K->512K(19456K) [PSPermGen: 2862K->2862K(21504K)], 0.0350643 secs] [Times: user=0.02 sys=0.00, real=0.03 secs]
解释：
日志最开始的GC和Full GC表示垃圾回收的停顿类型;
PSYoungGen中最前面的PS代表垃圾收集器是Parallel Scavenge收集器,回收的区域是新生代(YoungGen)
ParOldGen中最前面的Par代表垃圾收集器是Parallel Old收集器,回收的区域是老年代(OldGen).
方括号内的9216K->1024K(9216K)中9表示GC前该内存区域使用容量->GC后该内存区域已使用容量(该内存区域总容量).
1246196K->1246220K(1287040K)表示GC前Java堆已使用容量->GC后Java对已使用容量(Java对总容量).
0.2398360 secs表示GC所占有时间.

```

## 3.持久带溢出-OutOfMemoryError: PermGen space
### 3.1 解释
永久保存区域（permanent generation），它们不属于Java堆的一部分，用来存放类和类的描述（常量池，字段，方法数据等）。当永久保存区域的空间耗尽时OutOfMemoryError： PermGen space就会发生，这个错误一般是由于内存泄漏导致的。所谓内存泄漏，是指java类和类加载器在被取消部署后不能被垃圾回收。

### 3.2 解决方法
当遇到java.lang.OutOfMemoryError：PermGen space错误时，我们可以做的第一件事情是增加永久保存区域的最大尺寸，该尺寸的缺省设置是64 M，我们可以将它设置成128 M以上。通过-XX:PermSize和-XX:MaxPermSize来设置。

## 4.OutOfMemoryError-unable to create native thread
### 4.1 解释
- 《深入理解JAVA虚拟机》中有介绍：当给虚拟机分配的内存过大，会导致创建线程的时候需要的native内存太少。
- 操作系统对每个进程的内存是有限制的，我们启动Jvm,相当于启动了一个进程，假如我们一个进程占用了4G的内存，那么通过下面的公式计算出来的剩余内存就是建立线程栈的时候可以用的内存。 线程栈总可用内存=4G-（-Xmx的值）- （-XX:MaxPermSize的值）- 程序计数器占用的内存 通过上面的公式我们可以看出，-Xmx 和 MaxPermSize的值越大，那么留给线程栈可用的空间就越小，在-Xss参数配置的栈容量不变的情况下，可以创建的线程数也就越小。

> 注：-Xss128k:设置每个线程的堆栈大小.JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右.
### 4.2 解决办法
增大进程所占用的总内存，或者减少-Xmx或者-Xss来达到创建更多线程。
