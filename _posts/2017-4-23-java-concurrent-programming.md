---
layout: post
title: 《Java并发编程的艺术》笔记
categories: Java
excerpt: The Art of Java Concurrency Programming.<br><br><img src="http://upload-images.jianshu.io/upload_images/658453-a94405da52987372.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="70%">
---

> 好记性不如烂笔头。多读多思考。

### 基本概念 & Java 并发机制的底层实现原理

上下文切换：CPU在任务切换前会保存前一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次任务切换。

内存屏障：一组处理器指令，用于实现对内存操作的顺序限制。

#### 锁的升级

> 现在我们应该知道，Synchronized 是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的 Mutex Lock 来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么 Synchronized 效率低的原因。因此，这种**依赖于操作系统 Mutex Lock 所实现的锁我们称之为“重量级锁”**。JDK 中对 Synchronized 做的种种优化，其核心都是为了减少这种重量级锁的使用。JDK1.6 以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和“偏向锁”。

每一个线程在准备获取共享资源时： 
已经获取偏向锁的线程为线程1， 新线程为：线程2
第一步，线程2检查MarkWord里面是不是放的自己的ThreadId ,如果是，表示当前线程是处于 “偏向锁” ，就可以直接执行方法体了。
第二步，如果MarkWord不是自己的ThreadId, 用CAS来执行切换，如果不成功，线程2根据MarkWord里现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。 （线程1的同步体执行完后 会根据线程2的请求，暂停线程，置空markword里面的线程ID）
第三步，这样线程2就以轻量级的锁机制工作，如果这时线程3进入，就会进入自旋模式等待锁
第四步，自旋的线程3在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于 轻量级锁的状态，如果自旋失败 ，即自旋时间结束，仍然没有获取轻量级锁，进入重量级锁。
第五步，线程3进入重量级锁，将对象的markword修改为指向重量级锁的指针，线程2执行为同步体，修改Markword时，会失败，这样线程2就会意识到进入重量级锁了，
第六步，线程2释放锁，通知重量级锁唤醒阻塞队列。

> **轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能。**

处理器实现原子操作的方式：总线锁（锁住整个内存）；缓存锁（在处理器内部缓存中实现原子操作，使其他处理器不能缓存 i 的缓存行）。

Java 实现原子操作的方式：锁和循环 CAS（Compare and Swap 比较并交换）；CAS 利用了处理器的 CMPXCHG 指令（该指令是原子的）。

**除了偏向锁，JVM 实现锁的方式都用了循环 CAS**，即当一个线程想进入同步块的时候使用循环 CAS 的方式来获取锁，当它退出同步块的时候使用循环 CAS 释放锁。

```java
// 循环CAS
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
```

### Java内存模型

3个同步原语：synchronized，volatile，final；

并发编程的两个关键问题：线程间通信和线程间同步；

在**共享内存的并发模型**中，线程之间共享内存的公共状态，通过读-写内存的公共状态进行隐式通信。在**消息传递的并发模型**中，线程之间没有公共状态，必须通过发送消息来显式进行通信。

同步是指用于控制不同线程间操作发生相对顺序的机制。在共享内存并发模型里，同步是**显式进行**的——程序员需要显式指定某个方法或某段代码需要在线程间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是**隐式进行**的。

Java的并发采用的是共享内存模型，所以Java线程之间的通信总是隐式进行。

Java内存模型(JMM)：


![](http://upload-images.jianshu.io/upload_images/658453-026c3a7055adb24f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他硬件和编译器优化。（不完全是内存，也不完全是Cache））

从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤：

1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。
2. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量。


![](http://upload-images.jianshu.io/upload_images/658453-d3eb8ec1e8673d4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重要概念：**重排序**，编译器重排序和处理器重排序，为了提高并行度。

数据依赖：写后读，写后写，读后写；这3种情况，只要重排序两个操作的执行顺序，程序的执行结果就会改变；所以重排序时会遵守数据依赖性，不会改变存在数据依赖关系的两个操作的执行顺序。

控制依赖：由于处理器会采用**分支预测**技术来提高并行度，`i = a * a`可能会被重排序到`if (flag)`之前执行——这在单线程中是没问题的，但在多线程环境下就可能改变程序的执行结果。

```java
if (flag) {
    i = a * a;
}
```

as-if-serial语义：不管怎么重排序，单线程程序的执行结果不能被改变。

#### happens-before

JSR-133使用happens-before的概念来**阐述操作之间的内存可见性**。在JMM中，**如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before关系**。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。

happen-before的定义如下：

1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前 
2. 两个操作之间存在happens-before关系，**并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行**。如果重排序之后的执行结果与按照原来那种happens-before关系执行的结果一致，那么JMM允许编译器和处理器进行这种重排序

**as-if-serial语义保证单线程内的程序执行结果不会改变，happens-before保证正确同步的多线程程序的执行结果不会被改变。**

总共有六条规则：

* 程序顺序规则：一个线程中的每个操作，happens-before于随后该线程中的任意后续操作
* 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的获取
* volatile变量规则：对一个volatile域的写，happens-before于对这个变量的读
* 传递性：如果A happens-before B，B happens-before C，那么A happens-before C
* start规则：如果线程A执行线程B的start方法，那么线程A的ThreadB.start()happens-before于线程B的任意操作
* join规则：如果线程A执行线程B的join方法，那么线程B的任意操作happens-before于线程A从TreadB.join()方法成功返回。

顺序一致性内存模型：顺序一致性内存模型是一个被计算机科学家理想化了的**理论参考模型**，它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性：

1. 一个线程中的所有操作必须按照程序的顺序来执行。
2. （不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。

顺序一致性内存模型的视图：

![](http://upload-images.jianshu.io/upload_images/658453-ae47c5f0262b47c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在JMM中，临界区内的代码可以重排序，但不允许临界区内的代码“溢出”到临界区之外，那样会破坏监视器的内存语义。

JMM保证：单线程程序和正确同步的多线程程序的执行结果与在顺序一致性内存模型中的执行结果相同。

#### volatile

对volatile变量的单个读写，可以看成是使用了同一个锁对这些单个读写作了同步。（这样，即使是64位的long/double型变量，只要用volatile修饰，对该变量的读写就具有了原子性。注意，++这种复合操作依旧不具有原子性。）

volatile变量自身的特性：

* 可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
* 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

volatile的内存语义（对内存可见性的影响）

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。
* 当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保**volatile写之前的操作不会被编译器重排序到volatile写之后。**

当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保**volatile读之后的操作不会被编译器重排序到volatile读之前。**

**当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。**

#### 锁的内存语义

众所周知，锁可以让临界区互斥执行；但锁有一个同样重要，但常常被忽视的功能：锁的内存语义。

* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中
* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须要从主内存中去读取共享变量

对比锁释放-获取的内存语义与volatile写-读的内存语义，可以看出：锁释放与volatile写有相同的内存语义；锁获取与volatile读有相同的内存语义。

#### final的内存语义

* JMM禁止编译器把final域的写重排序到构造函数之外（**对普通域的写可能被重排序到构造函数之外！**）
* 在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作（这两个操作之间存在间接依赖，大多数处理器会遵守间接依赖，不会重排序这两个操作，但有少数处理器不遵守间接依赖关系，这个规则就是专门用来针对这种处理器的）

如果final域是引用类型：

```java
public class FinalReferenceExample {
    final int[] intArray;                     //final是引用类型
    static FinalReferenceExample obj;

    public FinalReferenceExample () {        //构造函数
        intArray = new int[1];              //1
        intArray[0] = 1;                   //2
    }

    public static void writerOne () {          //写线程A执行
        obj = new FinalReferenceExample ();  //3
    }
    ...
}
```

这里final域为一个引用类型，它引用一个int型的数组对象。对于引用类型，写final域的重排序规则对编译器和处理器**增加**了如下约束：

在构造函数内对一个**final引用的对象的成员域的写入**，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

在上图中，1是对final域的写入，2是对这个final域引用的对象的成员域的写入，3是把被构造的对象的引用赋值给某个引用变量。这里除了前面提到的1不能和3重排序外，**2和3也不能重排序**。

**为什么final引用不能从构造函数内“逸出”**

前面我们提到过，写final域的重排序规则可以确保：**在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化过了**（构造函数完成，对象引用才会产生）。其实要得到这个效果，还需要一个保证：在构造函数内部，不能让这个被构造对象的引用为其他线程可见，也就是对象引用不能在构造函数中“逸出”。为了说明问题，让我们来看下面示例代码：

```java
public class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;

    public FinalReferenceEscapeExample () {
        i = 1;                              //1 写final域
        obj = this;                          //2 this引用在此“逸出”
    }

    public static void writer() {
        new FinalReferenceEscapeExample ();
    }

    public static void reader {
        if (obj != null) {                     //3
            int temp = obj.i;                 //4
        }
    }
}
```

这里1和2可能会发生重排序，导致final域在被正确初始化之前对象引用就暴露了，从而在线程B的reader中访问到未初始化的final域。

JSR-133为什么要增强final的语义

在旧的Java内存模型中 ，最严重的一个缺陷就是**线程可能看到final域的值会改变**。比如，一个线程当前看到一个整形final域的值为0（还未初始化之前的默认值），过一段时间之后这个线程再去读这个final域的值时，却发现值变为了1（被某个线程初始化之后的值）。最常见的例子就是在旧的Java内存模型中，String的值可能会改变。

为了修补这个漏洞，JSR-133专家组增强了final的语义。通过为final域增加写和读重排序规则，可以为java程序员提供初始化安全保证：**只要对象是正确构造的（被构造对象的引用在构造函数中没有“逸出”），那么不需要使用同步（指lock和volatile的使用），就可以保证任意线程都能看到这个final域在构造函数中被初始化之后的值**。

#### 双重检查锁定与延迟初始化

延迟初始化：推迟一些高开销的对象初始化操作，并且只有在使用这些对象时才进行初始化。

```java
private static Instance instance;
public synchronized static Instance getInstance() {
    if (instance == null) {
        instance = new Instance();
    }
    return instance;
}
```

上面的方法虽然线程安全，但用synchronized将导致性能开销。

一个“聪明”的技巧：双重检查锁定：

```java
public class DoubleCheckLocking {
    private static Instance instance;
    public  static Instance getInstance() {
        if (instance == null) {
            synchronized(DoubleCheckLocking.class) {
                if (instance == null) {
                    instance = new Instance(); // 问题的根源出在这里
                }
            }
        }
        return instance;
    }
}

```

创建对象的过程instance = new Instance()可以分解为以下三步：
1. memory = allocate(); // 分配对象的内存空间
2. ctorInstance(memory); // 初始化对象
3. instance = memory; // 返回对象地址
4. 初次访问对象

其中，**2和3可能会被重排序**！重排序之后变成了：分配对象内存空间，返回对象地址，初始化对象；（在单线程内，只要保证2排在4的前面执行，单线程内的执行结果就不会被改变，这个重排序就是被允许的）

在多线程环境下，假设2和3发生重排序，那么一个未初始化的对象引用将从同步块中“**溢出**”，另一个线程可能会通过instance访问到这个未初始化的对象！

解决方案：

1，利用volatile的内存语义来禁止重排序

```java
private volatile static Instance instance;
```

根据volatile写的内存语义：volatile写之前的操作禁止被重排序到volatile写之后。这样上面2和3之间的重排序将会被禁止，问题根源得到解决。

2，利用类初始化的原子性

在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

```java
public class InstanceFactory {
    private static class InstanceHolder {
        public static Instance instance = new Instance();
    }
    public static Instance getInstance() {
        return InstanceHolder.instance ; // 这里将导致 InstanceHolder 类被初始化
    }
}
```


![](http://upload-images.jianshu.io/upload_images/658453-befd0517eb5512b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Java并发编程基础

设置线程优先级时，针对频繁阻塞（休眠或IO操作）的线程需要设置较高的优先级，而偏重计算的线程则设置较低的优先级，确保处理器不会被独占。

#### 线程状态变迁


![](http://upload-images.jianshu.io/upload_images/658453-9a0118108be70a9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可参考 [链接](http://blog.csdn.net/u010425776/article/details/54292463)

疑惑：貌似可以从等待态直接回到就绪/运行态，WHY / HOW？

另，书上一句话：

> 阻塞状态是线程阻塞在进入 synchronized 同步代码块或方法（获取锁）时的状态，**但是阻塞在 java.concurrent 包中 Lock 接口的线程状态却是等待状态**，因为 java.concurrent 包中的 Lock 接口对于阻塞的实现均使用了 LockSupport 类中的相关方法。

#### 中断

**中断可以理解为线程的一个标识位属性，它表示一个线程是否被其他线程进行了中断操作。**

调用一个线程对象的interrupt()方法，只是将该线程的中断标识位设为true，并不是真的“中断“了该线程。这个地方很容易迷惑人。

一个被中断的线程（被调用了interrupt()方法）**如何响应中断完全取决于该线程本身**。

线程有两种方法来判断自己是否被中断：
1. 实例方法isInterrupted()，返回true/false，不对中断标识位复位；
2. 静态方法Thread.interrupted()，返回true/false，同时对中断标识位进行复位；

Object.wait()，Thread.sleep()，Thread.join()等方法均声明抛出InterruptedException异常，说明这些方法是**可中断的**——这些方法在执行时会**不断轮询监听中断标识位**，当发现其为true时，会恢复中断标识位（即设为false），并抛出InterruptedException异常。

进入synchronized块和Lock.lock()等操作是不可被中断的（不抛出中断异常）。

**安全地终止线程**

轮询中断标识位，或另设一个标志：

```java
public class Runner implements Runnable {
    private volatile boolean on = true;
    private long i;
    @Override
    public void run() {
        while (on && !Thread.currentThread().isInterrupted()) {
            i++;
        }
        System.out.println("Count i = " + i);
    }
    public void cancel() {
        on = false;
    }
}

Runner one = new Runner();
Thread t1 = new Thread(one);
t1.start();
...
t1.interrupt();

Runner two = new Runner();
new Thread(two).start();
...
two.cancel();
```

#### 等待/通知机制

等待/通知的经典范式：

```java
synchronized(obj) {
    while(条件不满足) {
        obj.wait();
    }
    处理逻辑；
}

synchronized(obj) {
    改变条件；
    obj.notifyAll();
}
```

在while循环中判断条件并调用wait()是使用wait()的唯一正确方式——这样能保证线程在睡眠前后都会检查条件。

wait()返回的前提是当前线程获得锁；返回后从wait()处继续执行。

注意一点：wait()会使当前对象释放锁，notify() 和 notifyAll() 不会！

```java
synchronized(obj) {
    if (条件不满足) {
        obj.wait();
    }
    处理逻辑；
}
```

用 if 为什么错了呢？

wait()的线程被其他线程用notify()或notifyAll()唤醒后，是需要先获得锁的（毕竟你是在synchronized块里）；如果在被唤醒到获得锁的这段时间内，条件又被另一个线程改变了，而你获得锁并从wait()方法返回后，直接跳出了 if 的条件判断——这时条件是不满足的，于是产生了逻辑错误。所以，线程在睡眠前后都需要检查条件。

状态转换图

![](http://upload-images.jianshu.io/upload_images/658453-1d8ac82d80e62241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

线程调用wait()方法释放锁，进入等待队列，等待状态（WAITING）；被notify()/notifyAll()唤醒后，进入同步队列，变为阻塞状态（BLOCKING）；随后可再次获得锁并从wait()返回继续执行。

#### 管道输入/输出流

4种实现：PipedOutputStream, PipedInputStream, PipedReader, PipedWriter

```java
PipedWriter out = new PipedWriter();
PipedReader in = new PipedReader();
out.connect(in); // 将输入流和输出流进行连接，否则在使用时会抛出IOException；
```

#### ThreadLocal

在main线程中定义一个ThreadLocal对象，在各个线程中访问时，访问到的是各个线程独立的版本——并且是**独立初始化**的ThreadLocal对象。


默认情况下 initValue() 返回 null 。线程在没有调用 set 之前，第一次调用 get 的时候， get 方法会默认去调用 initValue 这个方法。所以如果没有覆写这个方法，可能导致 get 返回的是 null 。当然如果调用过 set 就不会有这种情况了。但是往往在多线程情况下我们不能保证每个线程的在调用 get 之前都调用了 set ，所以最好对 initValue 进行覆写，以免导致空指针异常。

```java
public class ConcurrentProgramming {
    public static ThreadLocal<Integer> threadLocalInt = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };
    // public static ThreadLocal<Integer> threadLocalInt = new ThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        // threadLocalInt.set(0);
        // System.out.println(threadLocalInt.get()); // 这里可以正常输出，因为在当前main线程中是先set，再get；
        for (int i = 0; i < 2; i++) {
            new Thread(new Worker()).start();
        }
    }
}

class Worker implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            // 但在这里就报空指针错了———— 所以，并不是共享的同一个ThreadLocal对象，而是每个线程new一个，对吗？
            ConcurrentProgramming.threadLocalInt.set(ConcurrentProgramming.threadLocalInt.get() + 1);
            System.out.println(Thread.currentThread().getName() + ": " + ConcurrentProgramming.threadLocalInt.get());
        }
    }
}

output:
Thread-0: 1
Thread-1: 1
Thread-1: 2
Thread-1: 3
Thread-1: 4
Thread-0: 2
Thread-0: 3
Thread-0: 4
Thread-0: 5
Thread-1: 5
```

注意代码中的注释部分。没有重写initialValue()时，在main中set(0)然后get，没有问题；但在另外两个线程中的get却报空指针异常——说明在main中set的值只在main线程中可见。

> This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, ***independently initialized*** copy of the variable.

—— 每个线程有自己的、独立初始化的变量拷贝。

所以，每个线程会独自new一个Threadlocal对象，只是共用了同一个变量名，或你写的ThreadLocal匿名内部类。

#### 等待超时模式

开发人员经常会遇到这样的方法调用场景：调用一个方法时等待一段时间，如果该方法在给定的时间段内能够得到结果，那么将立刻返回；反之，超时返回默认结果。

实现方式：在经典的等待/通知模型的加锁、条件循环、逻辑处理的基础上作出非常小的改动：

```java
public synchronized Object get(long mills) throws InterruptedException {
    long future = System.currentTimeMillis() + mills;
    long remaining = mills;
    while ((result == null) && remaining > 0) {
        wait(remaining);
        remaining = future - System.currentTimeMillis();
    }
    return result;
}
```

（数据库连接池示例、线程池示例 未）

### Java中的锁

#### Lock接口

* void lock() 获取锁,调用该方法当前线程将会获取锁，当锁获取后，该方法将返回。
* void lockInterruptibly() throws InterruptedException 可中断获取锁，与lock()方法不同之处在于该方法会响应中断，即在锁的获取过程中可以中断当前线程
* boolean tryLock() 尝试非阻塞的获取锁，调用该方法立即返回，true表示获取到锁
* boolean tryLock(long time,TimeUnit unit) throws InterruptedException 超时获取锁，以下情况会返回：时间内获取到了锁，时间内被中断，时间到了没有获取到锁。
* void unlock() 释放锁
* Condition newCondition() 获取等待通知组件

#### 队列同步器

队列同步器AbstractQueuedSynchronizer(AQS)是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。下图显示了java.concurrent包的实现示意图：


![](http://upload-images.jianshu.io/upload_images/658453-befc2906354c9e0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

队列同步器的实现依赖内部的同步队列来完成同步状态的管理。它是一个FIFO的双向队列，当线程获取同步状态失败时，同步器会将当前线程和等待状态等信息包装成一个节点并将其加入同步队列，同时会阻塞当前线程。当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。

![](http://upload-images.jianshu.io/upload_images/658453-406838652d2deb64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 共享式同步状态获取与释放

共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态。以文件的读写为例，如果一个程序在对文件进行读操作，那么这一时刻对于该文件的写操作均被阻塞，而读操作能够同时进行。写操作要求对资源的独占式访问，而读操作可以是共享式访问。

![](http://upload-images.jianshu.io/upload_images/658453-8dd5564138f841bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左半部分，共享式访问资源时，其他共享式的访问均被允许，而独占式访问被阻塞；右半部分是独占式访问资源时，同一时刻其他访问均被阻塞。

#### 重入锁 ReentrantLock

重入锁 ReentrantLock，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的**重复加锁**。除此之外，该锁的还支持获取锁时的公平和非公平性选择。

对于独占锁（Mutex），考虑如下场景：当一个线程调用Mutex的lock()方法获取锁之后，如果再次调用lock()方法，则**该线程将会被自己所阻塞**，原因是Mutex在实现tryAcquire(int acquires)方法时没有考虑占有锁的线程再次获取锁的场景，而在调用tryAcquire(int acquires)方法时返回了false，导致该线程被阻塞。简单地说，Mutex是一个不支持重进入的锁。

synchronized关键字隐式的支持重进入，比如一个synchronized修饰的递归方法，在方法执行时，执行线程在获取了锁之后仍能连续多次地获得该锁，而不像Mutex由于获取了锁，而在下一次获取锁时出现阻塞自己的情况。

ReentrantLock虽然没能像synchronized关键字一样支持隐式的重进入，但是在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。

**锁获取的公平性问题**

公平性与否是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该和锁的请求顺序一致，也就是FIFO。

非公平性锁可能使线程“饥饿”，当一个线程请求锁时，只要获取了同步状态即成功获取锁。在这个前提下，刚释放锁的线程再次获取同步状态的几率会非常大，使得其他线程只能在同步队列中等待。

非公平锁可能使线程“饥饿”，为什么它又被设定成默认的实现呢？非公平性锁模式下线程上下文切换的次数少，因此其性能开销更小。公平性锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换。非公平性锁虽然可能造成线程“饥饿”，但极少的线程切换，保证了其更大的吞吐量。

#### 读写锁

在Java并发包中常用的锁（如ReentrantLock），基本上都是**排他锁**，这些锁在同一时刻只允许一个线程进行访问，而读写锁在同一时刻可以允许**多个读线程访问**，但是在写线程访问时，所有的读线程和其他写线程均被阻塞。读写锁维护了一对锁，一个读锁和一个写锁，通过分离读锁和写锁，使得并发性相比一般的排他锁有了很大提升。

除了保证写操作对读操作的可见性以及并发性的提升之外，读写锁能够简化读写交互场景的编程方式。假设在程序中定义一个共享的数据结构用作缓存，它大部分时间提供读服务（例如：查询和搜索），而写操作占有的时间很少，但是写操作完成之后的更新需要对后续的读服务可见。

在没有读写锁支持的（Java 5 之前）时候，如果需要完成上述工作就要使用Java的等待通知机制，就是当写操作开始时，所有晚于写操作的读操作均会进入等待状态，只有写操作完成并进行通知之后，所有等待的读操作才能继续执行（写操作之间依靠synchronized关键字进行同步），这样做的目的是使读操作都能读取到正确的数据，而不会出现脏读。

改用读写锁实现上述功能，只需要在读操作时获取读锁，而写操作时获取写锁即可，当写锁被获取到时，后续（非当前写操作线程）的读写操作都会被阻塞，写锁释放之后，所有操作继续执行，编程方式相对于使用等待通知机制的实现方式而言，变得简单明了。

一般情况下，读写锁的性能都会比排它锁要好，因为大多数场景读是多于写的。在读多于写的情况下，读写锁能够提供比排它锁更好的并发性和吞吐量。Java并发包提供读写锁的实现是ReentrantReadWriteLock。

```java
ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
Lock r = rwl.readLock();
Lock w = rwl.writeLock();
```

#### Condition接口

任何一个Java对象，都拥有一组监视器方法，主要包括wait()、notify()、notifyAll()方法，这些方法与synchronized关键字配合使用可以实现等待/通知模式。Condition接口也提供类似的Object的监视器的方法，主要包括await()、signal()、signalAll()方法，这些方法与Lock锁配合使用也可以实现等待/通知模式。

相比Object实现的监视器方法，Condition接口的监视器方法具有一些Object所没有的特性：

* Condition接口可以支持多个等待队列：一个Lock实例可以绑定多个Condition。
* Condition接口支持在等待时**不响应中断**：wait()是会响应中断的；
* Condition接口支持**等待到将来的某个时间点返回**（和awaitNanos(long)/wait(long)不同！）：awaitUntil(Date deadline)；

```java
class BoundedBuffer {
    final Lock lock = new ReentrantLock();// 锁对象
    final Condition notFull = lock.newCondition(); //写线程条件
    final Condition notEmpty = lock.newCondition();//读线程条件
    final Object[] items = new Object[100];// 初始化一个长度为100的队列
    int putptr/* 写索引 */, takeptr/* 读索引 */, count/* 队列中存在的数据个数 */;

    public void put(Object x) throws InterruptedException {
        lock.lock(); //获取锁
        try {
            while (count == items.length)
                notFull.await();// 当计数器count等于队列的长度时，不能再插入，因此等待。阻塞写线程。
            items[putptr] = x;//赋值
            putptr++;

            if (putptr == items.length)
                putptr = 0;// 若写索引写到队列的最后一个位置了，将putptr置为0。
            count++; // 每放入一个对象就将计数器加1。
            notEmpty.signal(); // 一旦插入就唤醒取数据线程。
        } finally {
            lock.unlock(); // 最后释放锁
        }
    }

    public Object take() throws InterruptedException {
        lock.lock(); // 获取锁
        try {
            while (count == 0)
                notEmpty.await(); // 如果计数器等于0则等待，即阻塞读线程。
            Object x = items[takeptr]; // 取值
            takeptr++;
            if (takeptr == items.length)
                takeptr = 0; //若读锁应读到了队列的最后一个位置了，则读锁应置为0；即当takeptr达到队列长度时，从零开始取
            count++; // 每取一个将计数器减1。
            notFull.signal(); //枚取走一个就唤醒存线程。
            return x;
        } finally {
            lock.unlock();// 释放锁
        }
    }
}
```

上面用了两个Condition。（是不是很熟悉？王道，信号量，线程间同步）


**等待队列与同步队列**

![](http://upload-images.jianshu.io/upload_images/658453-5a5eabb5681594c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在Object的监视器模型上，一个对象拥有一个同步队列和一个等待队列，而并发包中的Lock（更确切的说是同步器）可以拥有一个同步队列和多个等待多列。

### Java并发容器和框架

#### ConcurrentHashMap

在并发环境下，HashMap的put操作会引起死循环。因为多线程会导致HashMap的Entry链表形成环形数据结构，使得Entry的next节点永远不为空。 

HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

**ConcurrentHashMap的锁分段技术**

HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，**首先将数据分成一段一段的存储，然后给每一段数据配一把锁**，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

![](http://upload-images.jianshu.io/upload_images/658453-2dc1c5f623b56a20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。

ConcurrentHashMap的get操作

Segment的get操作实现非常简单和高效。先经过一次再哈希，然后使用这个哈希值通过哈希运算定位到segment，再通过哈希算法定位到元素，代码如下：（两次哈希）

```java
public V get(Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).get(key, hash);
}
```

ConcurrentHashMap的Put操作

由于put方法里需要对共享变量进行写入操作，所以为了线程安全，在操作共享变量时必须得加锁。Put方法首先定位到Segment，然后在Segment里进行插入操作。插入操作需要经历两个步骤，第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步定位添加元素的位置然后放在HashEntry数组里。（扩容的时候首先会创建一个两倍于原容量的数组，然后将原数组里的元素进行再hash后插入到新的数组里。为了高效ConcurrentHashMap不会对整个容器进行扩容，而只对某个segment进行扩容）

ConcurrentHashMap的size操作

如果我们要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。Segment里的全局变量count是一个volatile变量，那么在多线程场景下，我们是不是直接把所有Segment的count相加就可以得到整个ConcurrentHashMap大小了呢？不是的，虽然相加时可以获取每个Segment的count的最新值，但是拿到之后可能累加前使用的count发生了变化，那么统计结果就不准了。所以最安全的做法，是在统计size的时候把所有Segment的put，remove和clean方法全部锁住，但是这种做法显然非常低效。

因为在累加count操作过程中，之前累加过的count发生变化的几率非常小，所以ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。

#### 并发队列：ConcurrentLinkedQueue

用非阻塞的循环CAS方式实现。

#### Java中的阻塞队列

阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。

插入和移除操作的四种处理方式

![](http://upload-images.jianshu.io/upload_images/658453-af043eac9471a600.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException(“Queue full”)异常。当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 。
* 返回特殊值：插入方法会返回是否成功，成功则返回true。移除方法，则是从队列里拿出一个元素，如果没有则返回null
* 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里take元素，队列也会阻塞消费者线程，直到队列可用。
* 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。

**Java里的阻塞队列**

* ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
* LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
* PriorityBlockingQueue ：一个支持优先级排序的**无界**阻塞队列。
* DelayQueue：一个使用优先级队列实现的无界阻塞队列；支持**延时获取元素**——在创建元素时可以指定多久才能从队列中取出当前元素；
* SynchronousQueue：一个**不存储元素的阻塞队列**——每一个put操作必须等待一个take操作；
* LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
* LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

```java
// 大小1000的、线程公平的阻塞队列；
// 传入了大小参数，这就叫有界；
ArrayBlockingQueue fairQueue = new ArrayBlockingQueue(1000, true);
```

阻塞队列的实现原理，见前面BoundedBuffer的代码。（一个队列，一个锁，两个Condition：notFull，notEmpty，等待通知模型）

#### Fork/Join框架

![](http://upload-images.jianshu.io/upload_images/658453-a1a50f5761b4ae3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与MapReduce一致的思想。

ForkJoinTask(抽象类)：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制。Fork/Join框架提供了以下两个子类：
* RecursiveAction：用于没有返回结果的任务。
* RecursiveTask ：用于有返回结果的任务。

```java
package com.xiao;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 2; // 阈值
    private int start;
    private int end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            //如果任务大于阀值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等待子任务执行完，并得到其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            //合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //生成一个计算任务，负责计算1+2+3+4
        CountTask task = new CountTask(1, 4);
        //执行一个任务
        Future result = forkJoinPool.submit(task);
        try {
            System.out.println(result.get());
        } catch (InterruptedException | ExecutionException e) {

        }
    }
}
```

**Fork/Join框架的实现原理**

ForkJoinPool由ForkJoinTask数组和ForkJoinWorkerThread数组组成，ForkJoinTask数组负责存放程序提交给ForkJoinPool的任务，而ForkJoinWorkerThread数组负责执行这些任务。（类似于线程池的实现）

### Java中的13个原子操作类

原子更新方式

* 原子更新基本类型
* 原子更新数组
* 原子更新引用
* 原子更新属性（字段）

1，原子更新基本类型

* AtomicBoolean ：原子更新布尔类型
* AtomicInteger： 原子更新整型
* AtomicLong: 原子更新长整型

2，原子更新数组

* AtomicIntegerArray ：原子更新整型数组里的元素
* AtomicLongArray :原子更新长整型数组里的元素
* AtomicReferenceArray : 原子更新引用类型数组的元素

3，原子更新引用类型

* AtomicReference ：原子更新引用类型
* AtomicReferenceFieldUpdater ：原子更新引用类型里的字段
* AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和应用类型

4，原子更新字段类

* AtomicIntegerFieldUpdater:原子更新整型的字段的更新器
* AtomicLongFieldUpdater：原子更新长整型字段的更新器
* AtomicStampedReference:原子更新带有版本号的引用类型。该类将整型数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。

（恩，是个坑，需要踩）

### Java中的并发工具类

#### CountDownLatch

（Latch：门闩）

用于等待其他线程完成操作。一个功能更强大的 join().

```java
CountDownLatch c = new CountDownLatch(2); // 等待两个[点]完成；
...
c.countDown(); // 第一个等待的操作完成；
...
c.countDown(); // 第二个等待的操作完成；

...
c.await(); // 等待两个操作完成；
...
```

CountDownLatch(N)等待N个点完成；这里说的N个点，可以是Ｎ个线程，也可以是一个线程里的Ｎ个执行步骤。

#### 同步屏障：CyclicBarrier

让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会打开，所有被屏障拦截的线程才会继续运行。

```java
CyclicBarrier c = new CyclicBarrier(2); // 屏障会拦截/等待两个线程；

// 在第一个线程中；
c.await(); // 当前线程（执行了某些操作后）到达屏障；

// 在第二个线程中；
c.await(); // 当前线程（执行了某些操作后）到达屏障；
```

**CyclicBarrier和CountDownLatch的区别**

CountDownLatch的计数器只能用一次，而CyclicBarrier的计数器可以使用reset()方法重置。所以CyclicBarrier可以处理更复杂的业务场景。例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。

#### 控制并发线程数的Semaphore

**信号量**，用来控制同时访问特定资源的线程数量。

```java
Semaphore s = new Semaphore(10);
Executor threadPool = Executors.newFixedThreadPool(30);

for (int i = 0; i < 30; i++) {
    threadPool.execute(new Runnable() {
        @Override
        public void run() {
            try {
                s.acquire();
                System.out.println("Save Date");
                s.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
}
```

在代码中，虽然有30个线程在执行，但只允许10个并发执行。

#### 线程间交换数据的Exchanger

Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange()方法，然后两个线程交换数据。

```java
Exchanger<String> exchanger = new Exchanger<>();
// 在线程A中；
try {
    String B = exchanger.exchange("A's data");
} catch (InterruptedException e) {
    e.printStackTrace();
}

// 在线程B中；
try {
    String A = exchanger.exchange("B's data");
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

### Java中的线程池

#### corePool

首先理解一个[corePool 核心池]的概念：核心池是一个线程池的基本/平均能力保障。在线程池的使用初期，随着任务的提交，线程池会先尽快填满核心池——**提交一个任务就创建一个线程，即使核心池中有空闲的线程**。如果线程池有温度的话，核心池就是线程池的“常温”。

![](http://upload-images.jianshu.io/upload_images/658453-88811e96d51a13b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/658453-93094229da3e79fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 线程池的创建

我们可以通过ThreadPoolExecutor来创建一个线程池。

```java
new ThreadPoolExecutor(corePoolSize, maximumPoolSize,
keepAliveTime, milliseconds,runnableTaskQueue, threadFactory,handler);
```

* corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，**即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建**。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
* runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。
    * ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
    * LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
    * SynchronousQueue：一个**不存储元素**的阻塞队列。每个插入操作(offer())必须等到另一个线程调用移除操作(poll())，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
    * PriorityBlockingQueue：一个具有优先级得无限阻塞队列。
maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
* ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。
* RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略。
    * AbortPolicy：直接抛出异常。
    * CallerRunsPolicy：只用调用者所在线程来运行任务。
    * DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
    * DiscardPolicy：不处理，丢弃掉。
当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。
* keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
* TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

**提交任务**

```java
void execute(Runnable command) // 没有返回值；
<T> Future<T> submit(Callable<T> task) // 有返回值的任务；
```

**关闭线程池**

我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，但是它们的实现原理不同，shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。

只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。

#### 合理的配置线程池

要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：

1. 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
2. 任务的优先级：高，中和低。
3. 任务的执行时间：长，中和短。
4. 任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务配置尽可能少的线程数量，如配置Ncpu+1个线程的线程池。IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。我们可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。

**线程池的监控**

通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

* taskCount：线程池需要执行的任务数量。
* completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
* largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
* getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减。
* getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。通过继承线程池并重写线程池的beforeExecute，afterExecute和terminated方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。

### Executor框架

#### Executor框架的结构和成员

![](http://upload-images.jianshu.io/upload_images/658453-de5105afbbb6594c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Executor框架主要由3大部分组成如下：

1. 任务：Runnable接口和Callable接口；
2. 任务的执行：Executor接口，继承Executor的ExecutorService接口，以及ExecutorService接口的两个实现类ThreadPoolExecutor和ScheduledThreadPoolExecutor；以及一个工具类：Executors；
3. 异步计算的结果：Future接口和Future接口的实现类FutureTask；

#### ThreadPoolExecutor

ThreadPoolExecutor通常由工厂类Executors来创建。Executors可以创建3种类型的ThreadPoolExecutor：SingleThreadExecutor，FixedThreadPool，CachedThreadPool；

FixedThreadPool是使用固定线程数的线程池,Executors提供的API有如下两个：

1. `public static ExecutorService newFixedThreadPool(int nThreads);`
2. `public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory theadFactory);`

```java
public static ExecutorService newFixedThreadPool(int nThreads){
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
// corePoolSize和maximumPoolSize都设为nThreads；
// 空闲线程的存活时间为0，意味着多余的空闲线程会立即死亡；
// 使用无界的LinkedBlockingQueue，不会拒绝任务；
```

FixedThreadPool满足了资源管理的需求，可以限制当前线程数量。适用于负载较重的服务器环境。

SingleThreadExecutor使用单线程执行任务，Executors提供的API有如下两个:

1. `public static ExecutorService newSingleThreadExecutor();`
2. `public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory);`

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
// corePoolSize和maximumPoolSize均为1；
// 多余的空闲线程立即死亡；
// 不拒绝任务；
```

SingleThreadExecutor保证了任务执行的顺序，不会存在多线程活动。

CachedThreadPool是无界线程池，Executors提供的API有如下两个：

1. `public static ExecutorService newCachedThreadPool();`
2. `public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory);`

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
// 线程池大小不限；
// 多余的空闲线程存活60s;
// 使用不存储元素的SynchronousQueue作为线程池的任务队列，一个offer操作必须等待另一个线程的poll操作；如果主线程提交任务的速度高于线程池中处理任务的速度，CachedThreadPool会不断创建新线程；极端情况下，可能会因为创建过多的线程而耗尽CPU和内存资源；
```

CachedThreadPool适用于执行很多短期异步任务的小程序，适用于负载较轻的服务器。

#### ScheduledThreadPoolExecutor

它是ThreadPoolExecutor的子类且实现了ScheduledExecutorService接口，它可以在给定的延迟时间后执行命令，或者定期执行命令，它比Timer更强大更灵活。

Executors可以创建的ScheduledThreadPoolExecutor的类型有ScheduledThreadPoolExecutor和SingleThreadScheduledExecutor等

ScheduledThreadPoolExecutor具有固定线程个数，适用于需要多个后台线程执行周期任务，并且为了满足资源管理需求而限制后台线程数量的场景,Executors中提供的API有如下两个：

1. `public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize);`
2. `public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory);`
 

SingleThreadScheduledExecutor具有单个线程，Executors提供的创建API有如下两个：

1. `public static ScheduledExecutorService newSingleThreadScheduledExecutor();`
2. `public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory);`

它适用于单个后台线程执行周期任务，并且保证顺序一致执行的场景。

#### ScheduledThreadPoolExecutor

在给定延迟之后执行任务，或者定期执行任务。ScheduledThreadPoolExecutor的功能与Timer类似，但更强大、更灵活。Timer对应的是单个后台线程，而ScheduledThreadPoolExecutor可以在构造函数中指定多个对应的后台线程数。

![P70421-224830(1).jpg](http://upload-images.jianshu.io/upload_images/658453-9103be6114583660.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ScheduledThreadPoolExecutor中线程执行某个周期任务的4个步骤：

步骤1：线程1从工作队列DelayQueue中获取已到期的task；
步骤2：线程1执行该task；
步骤3：线程1修改ScheduledFutureTask的time变量为下次被执行的时间；
步骤4：线程1将修改后的task重新放回DelayQueue中。

#### FutureTask类

Runnable接口：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

Callable接口（可以有返回值，可以抛出异常）：

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

Future接口：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

FutureTask类的构造方法：

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

ExecutorService的3个submit()方法都返回Future：

```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result); // 执行成功返回指定的值result；
Future<?> submit(Runnable task); // 线程执行成功返回null；
```

Callable和Future的普通用法：

```java
Callable<Integer> callable = new Callable<Integer>() {
    public Integer call() throws Exception {
        return new Random().nextInt(100);
    }
};
FutureTask<Integer> future = new FutureTask<Integer>(callable);
new Thread(future).start();
int result = future.get();
```

#### Executors

![](http://upload-images.jianshu.io/upload_images/658453-a745331000b24842.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
