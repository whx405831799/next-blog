title: 多线程必知必会源码系列-Semaphore信号量
date: 2017/05/14 17:15
categories:
- 多线程
tags:
- 多线程
- 源码
---
## 1.Semaphpre信号量
### 1.1 定义
信号量用来控制同时访问某个特定资源的操作数量。还可以用来实现某种资源池，或者对容器施加边界。Semaphore可以控制某个资源可被同时访问的个数。
### 1.1 主要方法
- public void acquire()  从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
- public void acquire(int permits)从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。
- public void release() 释放一个许可，将其返回给信号量。
- public void release(int permits)释放给定数目的许可，将其返回到信号量。

### 1.2 例子用法

```
package concurrent;

import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

/**
 * Created by Administrator on 2017/5/11.
 * 总共十个人吃饭，但是只有3个位置吃饭，需要等吃完其他人才能去吃
 */
public class TestSemaphore {
    //信号量设置3个，也就是同时只能有三个吃饭
    final Semaphore semaphore = new Semaphore(3);

    //吃饭的任务
    class  DinnerTask implements Runnable{
        private int i;
        public DinnerTask(int i) {
            this.i = i;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("我是无名" + i + "，正在吃饭！现在可利用的信号量数量为" + semaphore.availablePermits());
                //Thread.currentThread().sleep(1000);
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TestSemaphore testSemaphore = new TestSemaphore();
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (int i = 1 ;i <= 10 ;i++){
            executor.submit(testSemaphore.new DinnerTask(i));
        }
        Thread.currentThread().sleep(3000);
        System.out.println("现在可利用的信号量为" + testSemaphore.semaphore.availablePermits());

    }

}



结果为：
我是无名2，正在吃饭！现在可利用的信号量数量为2
我是无名6，正在吃饭！现在可利用的信号量数量为2
我是无名10，正在吃饭！现在可利用的信号量数量为2
我是无名3，正在吃饭！现在可利用的信号量数量为2
我是无名7，正在吃饭！现在可利用的信号量数量为2
我是无名4，正在吃饭！现在可利用的信号量数量为2
我是无名8，正在吃饭！现在可利用的信号量数量为2
我是无名1，正在吃饭！现在可利用的信号量数量为2
我是无名5，正在吃饭！现在可利用的信号量数量为2
我是无名9，正在吃饭！现在可利用的信号量数量为2
现在可利用的信号量为3
```


## 2.源码分析
会罗列出方法及关键代码解析。
### 2.1 所有方法
#### 2.1.1 构造方法
- Semaphore(int permits)  创建具有给定的许可数和非公平的公平设置的 Semaphore。
- Semaphore(int permits, boolean fair) 
创建具有给定的许可数和给定的公平设置的 Semaphore。

#### 2.1.2 方法
- 1.1中的四个方法就不重复写了。
- public void acquireUninterruptibly()从此信号量中获取许可，在有可用的许可前将其阻塞。
- public void acquireUninterruptibly(int permits)从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞。
- public int availablePermits()返回此信号量中当前可用的许可数。
- public int drainPermits()获取并返回立即可用的所有许可。
- protected Collection<Thread> getQueuedThreads()返回一个 collection，包含可能等待获取的线程。
- public final int getQueueLength()返回正在等待获取的线程的估计数目。
- public final boolean hasQueuedThreads()查询是否有线程正在等待获取。
- public boolean isFair()返回此信号量公平设置的状态。
- protected void reducePermits根据指定的缩减量减小可用许可的数目。
- public String toString()返回标识此信号量的字符串，以及信号量的状态。
- public boolean tryAcquire()仅在调用时此信号量存在一个可用许可，才从信号量获取许可。
- public boolean tryAcquire(int permits)仅在调用时此信号量中有给定数目的许可时，才从此信号量中获取这些许可。
- public boolean tryAcquire(long timeout, TimeUnit unit)如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。
- public boolean tryAcquire(int permits, long timeout, TimeUnit unit)如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。

### 2.2 关键源码解析
#### 2.2.1 acquire方法获取许可
acquire方法，实际上调用了sync的acquireSharedInterruptibly()方法。

```
private final Sync sync;
...
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
接着看Sync类的acquireSharedInterruptibly()方法，Sync是Semaphore的静态内部类，Sync继承了AbstractQueuedSynchronizer。acquireSharedInterruptibly()方法实际上是AQS的方法。

```
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    //如果线程状态是中断状态，则抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    //获取共享锁，如果获取失败则调用doAcquireSharedInterruptibly
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```
接着看tryAcquireShared()方法。这里的调用会分成公平模式和非公平模式，不同的模式会有不同的实现。当你实例化Semaphore时，会调用对应的构造方法，默认的构造方法是非公平模式。

```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}
```
我们先来看**非公平模式**的tryAcquireShared()。实际上是调用了NonfairSync父类Sync的nonfairTryAcquireShared方法。

```
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;

    NonfairSync(int permits) {
        super(permits);
    }
    //实际上是去调用了父类Sync的nonfairTryAcquireShared方法
    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}
```
Sync的nonfairTryAcquireShared方法。getState是一个许可数的方法。在Semaphore的构造方法中，new Sync时，会调用AQS中的setState方法设置volatile变量state为许可数。getState则可以获取当前的信号量许可数。
```
final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            //获取信号量的许可数
            int available = getState();
            //remaining为目前可用许可数减去当前要申请的许可数
            int remaining = available - acquires;
            //如果remaining小于0或者remaining大于0的同时compareAndSetState方法返回true，则直接返回remaining,然后当remaining<0调用acquireSharedInterruptibly()方法中的doAcquireSharedInterruptibly()方法处理,当remaining大于0则代表获取信号量成功;
            //如果remaining大于等于0而且compareAndSetState()返回false，则继续循环。
            //compareAndSetState方法实际上是采用CAS去更新state值，最终会调用一个compareAndSwapInt的native方法。当更新成功则返回true。
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
```
上面描述及注释中说到了获取信号量成功的情况，下面看看当remaining返回小于0的情况。这时会调用acquireSharedInterruptibly()的doAcquireSharedInterruptibly()方法，此方法也是AbstractQueuedSynchronizer种的方法。此方法的作用应该是要将当前线程放入到等待队列中去。

为了理解下面代码，可以先看这里的AQS队列的介绍，再去看代码。参考网上AQS介绍，可以知道：AQS是一个双向链表，通过节点里的next指向当前节点后一个节点，prev变量指向当前节点前一个节点。每个节点中都包含了一个线程和一个类型变量（用于识别节点是独占节点还是共享节点。btw，ReadWriteLock是共享锁的实现，ReentrantLock是独占锁的实现）。头节点中的线程为目前正在占有锁的线程，其他节点的线程表示为正在等待获取锁的线程。如下图所示：
![image](http://ohoyqlwj0.bkt.clouddn.com/AQS%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8.png)
头节点，表示正在占有锁的节点，其他的节点（node1,node2）为正在等待获取锁的节点，它们通过next、pre变量指向前后节点，形成了AQS中的双向链表。 

```
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    //创建共享锁类型的节点，并通过addWaiter方法添加到队列尾部
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            //获取prev节点
            final Node p = node.predecessor();
            if (p == head) {
                //如果p为头结点，则去尝试获取锁
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //获取成功后设置头结点并唤醒其他节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            //如果当前节点上一个节点不是头结点，则会调用shouldParkAfterFailedAcquire方法设置此节点的prev节点的waitStatus状态为SIGNAL，代表后续节点都需要唤醒
            //parkAndCheckInterrupt会通过park方法阻塞当前线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}


private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```
上面可能会有两个疑问点，setHeadAndPropagate()方法怎么处理的以及shouldParkAfterFailedAcquir()怎么设置node的waitStatus状态。我们先来看看Node类的一些变量及shouldParkAfterFailedAcquir()方法、parkAndCheckInterrupt()方法的代码。

```
static final class Node {
  ...
  /** 此状态代表线程被取消 */
  static final int CANCELLED =  1;
  /** 此状态代表后续节点处于等待状态，需要被唤醒 */
  static final int SIGNAL    = -1;
  /** 节点在等待队列中，节点线程等待在Condition上，
    *当其他线程对Condition调用了signal()方法后，该节点将会 
    *从等待队里中转移到同步队列中，加入对同步状态的获取
    */
  static final int CONDITION = -2;
  /**
   * 表示下一次共享式同步状态获取将会无条件地被传播下去 
   */
  static final int PROPAGATE = -3;
  /**
   * 状态值可以为CANCELLED、SINGNAL、CONDITION、PROPAGATE、0 
   */
  volatile int waitStatus;
  ...
  }

```

```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //获取prev节点的waitStatus状态
    int ws = pred.waitStatus;
    //如果为SIGNAL,直接返回
    if (ws == Node.SIGNAL)
        return true;
    //如果大于0，则为取消状态，则会把当前节点的prev节点移除，把当前节点的prev换成更前一个节点
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        //如果状态为0或者PROPAGATE，则通过CAS替换状态为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}


private final boolean parkAndCheckInterrupt() {
    //把线程置于waiting状态
    LockSupport.park(this);
    return Thread.interrupted();
}
```
上面的代码是在当节点的prev节点不为头结点时调用的，接下来我们再看一下当prev节点为头结点调用的setHeadAndPropagate()方法。主要是为了把当前节点设置为头结点，然后释放共享锁，唤醒unpack后续线程。

```
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node); //设置node节点为头结点
    /*
     * unnecessary wake-ups, but only when there are multiple
     * racing acquires/releases, so most need signals now or soon
     * anyway.
     * 其实这里我有一点不明白为什么h可能为null？
     */
    if (propagate > 0 || h == null || h.waitStatus < 0) {
        Node s = node.next;
        //当next节点为空或者为共享模式的节点，则执行doReleaseShared
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```
zhihou 我们看一下doReleaseShared()方法。当头结点的waitStatus为SIGNAL时，会通过CAS设置头结点的为0.设置成功后，将调用unparkSuccessor()，目的是找到后续节点，然后唤醒。

```
private void doReleaseShared() {
    /*
     * fails, if so rechecking.
     */
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                //找到后续节点，然后唤醒
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```
最后就看一下unparkSuccessor()方法，主要是找到头结点的后续节点，直到找到一个waitStatus不大于0，也就是非取消状态的节点，然后通过unpark唤醒此节点的线程。

```
private void unparkSuccessor(Node node) {
        //获取头节点状态
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        //寻找头结点的后续节点，当遇到取消状态的节点就略过继续找下一个节点
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            //唤醒此节点的线程
            LockSupport.unpark(s.thread);
    }
```

说到这里，就说完了非公平模式的的tryAcquireShared()及后续过程；那公平模式的区别是什么呢？区别只在这里，公平模式的tryAcquireShared()中会有一个判断当前线程是不是在AQS队列的头部的next节点的方法，如果此线程不在头部的next节点，则会交由上文中的doAcquireSharedInterruptibly()方法直接放入队列尾部。我们来看看公平模式的tryAcquireShared方法，多了个hasQueuedPredecessors()判断。
```
protected int tryAcquireShared(int acquires) {
        for (;;) {
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
    
 public final boolean hasQueuedPredecessors() {
        //目的就是判断是否是头结点的next节点的线程
        Node t = tail;  
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```
那么acquire方法就介绍完了。

#### 2.2.2 release方法释放许可


释放就没有分为公平和非公平模式了。
先来看第一个方法release(),实际上是调用了Sync的父类AbstractQueuedSynchronizer的releaseShared()方法。

```
public void release() {
    sync.releaseShared(1);
}
```
我们来看看releaseShared()方法，先去调用了tryReleaseShared()尝试释放锁。如果成功，则去做和申请许可中一样的doReleaseShared()方法去唤醒后续线程。

```
public final boolean releaseShared(int arg) {
    //尝试释放信号量
    if (tryReleaseShared(arg)) {
    //释放成功则去唤醒后续线程
        doReleaseShared();
        return true;
    }
    return false;
}
```
doReleaseShared()在上文中已经介绍过，接下来只看看tryReleaseShared方法。这个是由Sync重写了。方法目的是释放信号量。

```
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        //获取当前可用信号量
        int current = getState();
        //加上当前要释放的信号量
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        //通过CAS去设置新的可用信号量数量
        if (compareAndSetState(current, next))
            return true;
    }
}
```
release()方法也介绍完了。

## 3.总结
本文主要讲解了信号量的常用方法及举例，然后分析了acquire()和release()的源码。有些地方可能分析比较单薄，还需要经常回顾加深理解。






















