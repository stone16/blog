---
title: Java 多线程 基础知识(一)
date: 2020-04-30 19:36:12
categories: BackEnd
tags:
    - Java
    - Multi-threading
top:
---
# 1. 多线程知识基础

## 1.1 线程 vs 进程
进程是程序的一次执行过程，java当中，启动main函数就是启动了一个JVM进程，main函数所在的线程是其中之一，也称为主线程。

线程是比进程更小的执行单位，一个进程执行过程当中可以产生多个线程。同类的多个线程互相之间共享进程的堆和方法区的资源。每个线程有自己的程序计数器，虚拟机栈，和本地方法栈

线程与进程之间的关系如下图所示

![线程进程关系.png](https://i.loli.net/2020/04/30/3AwhmaPDLkcXKMb.png) 

运行时堆和方法区，还有常量是共享的。

**线程私有的**

+ 程序计数器
    + 当前线程所执行的字节码的行号指示器
    + 字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成
    + 为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存
    + 程序计数器是唯一不会出现OutOfMemoryError的内存区域，声明周期是完全跟着线程的，线程创建即创建，线程结束即结束
    + 程序计数器的私有主要是为了线程切换以后能够恢复到正确的执行位置上
+ 虚拟机栈
    + 描述java方法执行的内存模型
    + Java虚拟机栈由一个个栈帧组成，每个栈帧都拥有
        + 局部变量表
            + 存放了编译器已知的各种数据类型
            + 对象引用
        + 操作数栈
        + 动态链接
        + 方法出口信息
+ 本地方法栈
    + 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务
    + 本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

**线程共享资源**

+ 堆 
    + 存放对象实例
    + 几乎所有对象实例以及数组都在这里分配内存
    + 堆是Java垃圾收集器的主要区域，因此也被称作GC堆 Garbage Collected Heap 
    + 堆会根据时间的长度分为多种空间，来方便垃圾收集器来更好的回收内存
    + 还包括运行时常量池
+ 方法区
    + 用于存储已被虚拟机加载的类的信息，常量，静态变量，即时编译器编译后的代码等数据
    
+ 直接内存

## 1.2 并发 vs 并行

并发 - 同一时间段的多个任务都在执行
并行 - 单位时间内，多个任务同时执行

## 1.3 多线程带来的改变 

### 1.3.1 优势
+ 线程，是程序执行的最小单位，线程间的切换和调度成本远远小于进程。而且多核CPU时代意味着多个线程同时进行，这减少了线程上下文切换的开销
+ 多线程并发变成是开发高并发系统的基础
+ 单核时代多线程做的优化更多是提高CPU和IO的综合利用率。在一个线程做IO的时候，另外一个线程可以到内核当中做利用CPU的大量计算
+ 多核时代想做的事情就是同时使用多个内核一起来做这件事，提高整个运行的效率

### 1.3.2 可能的问题
需要解决内存泄漏，死锁，线程不安全相关的问题

+ 死锁
    + 多个线程被同时阻塞，在等待某个资源的释放
    + 产生死锁的条件
        + 互斥条件 -- 该资源任意时刻只由一个线程占用
        + 请求与保持条件 -- 一个进程因请求资源而阻塞时，对已获得的资源保持不放 
        + 不剥夺条件 -- 线程已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源
        + 循环等待条件 -- 若干进程之间形成一种头尾相接的循环等待资源关系


    public class DeadLockDemo {
        private static Object resource1 = new Object();//资源 1
        private static Object resource2 = new Object();//资源 2

        public static void main(String[] args) {
            new Thread(() -> {
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + "waiting get resource2");
                    synchronized (resource2) {
                        System.out.println(Thread.currentThread() + "get resource2");
                    }
                }
            }, "线程 1").start();

            new Thread(() -> {
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + "waiting get resource1");
                    synchronized (resource1) {
                        System.out.println(Thread.currentThread() + "get resource1");
                    }
                }
            }, "线程 2").start();
        }
    }

## 1.4 sleep（）vs wait（）
+ 最主要的区别在于sleep并没有释放锁，而wait方法会释放锁
+ 二者都暂停了当前线程的执行
+ wait用于线程之间的交互和通信，sleep用于暂停执行
+ wait()方法被调用以后，线程不会自动苏醒，需要别的线程调用同一个对象上的notify()或者notifyAll()方法

## 1.5 并发编程的重要特性
+ 原子性
    + 被修饰的代码块要不全都执行，要不都不执行，synchronized可以保证代码片段的原子性
+ 可见性
    + 当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改以后的新值的。volatile关键字可以保证共享变量的可见性
+ 有序性
    + Java 在编译器以及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。volatile 关键字可以禁止指令进行重排序优化   

## 1.6 volatile关键字
### 1.6.1 General

在当前的 Java 内存模型下，线程可以把变量保存本地内存（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。要解决这个问题，需要将变量声明为volatile，这就指示JVM，这个变量时不稳定的，每次使用它都到主存当中进行读取。
即volatile关键字可以起到：
+ 保证变量的可见性，从主内存当中拿数据
+ 防止指令重排序


## 1.7 synchronized关键字
### 1.7.1 General 
+ 解决多个线程之间访问资源的同步性问题
+ 可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行
+ 最开始是基于操作系统的mutex lock来实现，因为有用户态和内核态的转换，需要相对比较长的时间，性能不好；JDK 1.6之后做了大量优化，性能有了不小的提升
    + 自旋锁
    + 适应性自旋锁
    + 锁消除
    + 锁粗化
    + 偏向锁
    + 轻量级锁

### 1.7.2 使用方式

+ 修饰实例方法
    + 作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁 
+ 修饰静态方法
    + 给当前类加锁，会作用于类的所有对象实例
+ 修饰代码块
    + 指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁  


    // 线程安全的单例模式的实现
    public class Singleton {

        private volatile static Singleton uniqueInstance;

        private Singleton() {
        }

        public static Singleton getUniqueInstance() {
           //先判断对象是否已经实例过，没有实例化过才进入加锁代码
            if (uniqueInstance == null) {
                //类对象加锁
                synchronized (Singleton.class) {
                    if (uniqueInstance == null) {
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }

### 1.7.3 底层实现方式

+ 同步语句块的时候
    + 使用的是monitorenter和monitorexit指令
    + monitorenter指令指向同步代码块的开始位置
    + monitorexit指令指向同步代码块的结束位置
    + 锁计数器为0时可以获取，释放锁的时候再置为0
+ 同步方法的时候
    + 使用的是ACC_SYNCHRONIZED标识，指明该方法为一个同步方法，从而执行相应的同步调用

### 1.7.4 synchronized 关键字的底层优化

锁主要存在四中状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率

+ 偏向锁
    + 为了在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗
    + 偏向锁在无竞争的情况下会把整个同步都消除掉
    + 但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁 

+ 轻量级锁
    + 偏向锁失败的情况下会首先升级为轻量级锁
    + 在没有多线程竞争的前提下，减少传统重量级锁使用操作系统互斥量产生的性能消耗
    + 使用轻量级锁，不需要申请互斥量。另外，轻量级锁的加锁和解锁都用到了CAS操作 
    + 轻量级锁能够提升程序同步性能的依据是“对于绝大部分锁，在整个同步周期内都是不存在竞争的”，这是一个经验数据。如果没有竞争，轻量级锁使用 CAS 操作避免了使用互斥操作的开销。但如果存在锁竞争，除了互斥量开销外，还会额外发生CAS操作，因此在有锁竞争的情况下，轻量级锁比传统的重量级锁更慢！如果锁竞争激烈，那么轻量级将很快膨胀为重量级锁！

+ 自旋锁和自适应自旋
    + 互斥同步对性能最大的影响就是阻塞的实现，因为挂起线程/恢复线程的操作都需要转入内核态中完成
    + 般线程持有锁的时间都不是太长，所以仅仅为了这一点时间去挂起线程/恢复线程是得不偿失的
    + 自旋锁就是让线程执行一个忙循环，自旋锁和互斥锁不同之处在于不会休眠，调用者一直在那里循环看锁的保持者是否释放了锁
+ 锁消除
    + 编译器在运行的时候，如果检测到共享数据不可能存在竞争，就执行锁消除，以节省毫无意义的请求锁的时间

### 1.7.5 synchronized关键字和volatile关键字的区别

+ volatile是线程同步的轻量级实现，因此volatile性能会比synchronized关键字好。
+ 多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会
+ volatile关键字能保证数据的可见性，不能保证数据的原子性。synchronized都可以保证
+ volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性

# 2. ThreadLocal
## 2.1 ThreadLocal
通常情况下，我们创建的变量时可以被任何一个线程访问并修改的，如果想实现每一个线程都有自己的专属的本地变量的话，可以使用JDK提供的ThreadLocal类。ThreadLocal类解决的问题就是想让每个线程都绑定自己的值。

当我们创建了ThreadLocal变量，访问这个变量的每个线程都会有这个变量的本地副本，使用get()以及set()方法获取默认值或者将指改为当前线程所存的副本的值，从而避免线程安全问题。

    import java.text.SimpleDateFormat;
    import java.util.Random;

    public class ThreadLocalExample implements Runnable{

         // SimpleDateFormat 不是线程安全的，所以每个线程都要有自己独立的副本
        private static final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyyMMdd HHmm"));

        public static void main(String[] args) throws InterruptedException {
            ThreadLocalExample obj = new ThreadLocalExample();
            for(int i=0 ; i<10; i++){
                Thread t = new Thread(obj, ""+i);
                Thread.sleep(new Random().nextInt(1000));
                t.start();
            }
        }

        @Override
        public void run() {
            System.out.println("Thread Name= "+Thread.currentThread().getName()+" default Formatter = "+formatter.get().toPattern());
            try {
                Thread.sleep(new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //formatter pattern is changed here by thread, but it won't reflect to other threads
            formatter.set(new SimpleDateFormat());

            System.out.println("Thread Name= "+Thread.currentThread().getName()+" formatter = "+formatter.get().toPattern());
        }

    }


从ThreadLocal原理上来讲

    public class Thread implements Runnable {

         ......
        //与此线程有关的ThreadLocal值。由ThreadLocal类维护
        ThreadLocal.ThreadLocalMap threadLocals = null;

        //与此线程有关的InheritableThreadLocal值。由InheritableThreadLocal类维护
        ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
         ......
    }

+ Thread类中有一个threadLocals和一个inheritableThreadLocals变量，都是由ThreadLocalMap来实现的。默认情况下两个变量的值均为null，只有当前线程调用ThreadLocal类的set/ get方法时才创建他们

+ ThreadLocal类的set()方法



        public void set(T value) {
            Thread t = Thread.currentThread();
            ThreadLocalMap map = getMap(t);
            if (map != null)
                map.set(this, value);
            else
                createMap(t, value);
        }
        ThreadLocalMap getMap(Thread t) {
            return t.threadLocals;
        }
+ 从上面的代码中，可以看出最终的变量是放在了当前线程的ThreadLocalMap当中，并不是直接存在ThreadLocal上，ThreadLocal可以理解为只是ThreadLocalMap的封装，传递了变量值。
+ ThreadLocal类可以通过Thread.currentThread()获取当前线程对象之后，直接通过getMap(Thread t) 访问到该线程的ThreadLocalMap对象

# 3. AQS

AQS全称为AbstractQueuedSynchronizer，是一个用来构建锁和同步器的框架。

## 3.1 原理分析

核心思想为如果被请求的共享资源是空闲的，就将现在请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的资源被占用了，那么就需要一整套线程阻塞等待以及被唤醒时锁分配的机制，AQS使用CLH队列锁实现，将暂时获取不到锁的线程加入到队列当中。

CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配

+ AQS使用int变量来表示同步状态
+ 通过内置的FIFO队列来完成获取资源线程的排队工作
+ AQS使用CAS对该同步状态进行原子操作实现对其值的修改


    private volatile int state;//共享变量，使用volatile修饰保证线程可见性

    //返回同步状态的当前值
    protected final int getState() {  
            return state;
    }
     // 设置同步状态的值
    protected final void setState(int newState) { 
            state = newState;
    }
    //原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
    protected final boolean compareAndSetState(int expect, int update) {
            return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }

## 3.2 AQS资源共享方式

+ 共有两种资源共享方式
    + Exclusive 独占 
        + 只有一个线程能执行，又分为公平锁和非公平锁
        + 公平锁
            + 按照线程在队列中的排队顺序，先到者先拿到锁 
        + 非公平锁  
            + 当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
    + Share 共享
        + 多个线程可同时执行
            + semaphore
            + coutDownLatch
           
## 3.3 常用组件

+ Semaphore 信号量 -- 允许多个线程同时访问
    + synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。
+ CountDownLatch 倒计时器
    + CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行

+ CyclicBarrier 循环栅栏
    + CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）
    + 它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await()方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞
# 4. Atomic原子类

在这里指的是不可中断的操作，即便是多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。原子类体系就是就有原子/原子操作特征的类。


## 4.1 General

+ 基本类型 - 使用原子的方式更新基本类型
    +  AtomicInteger 整形原子类
    +  AtomicLong 长整形原子类
    +  AtomicBoolean 布尔型原子类
+ 数组类型
    + AtomicIntegerArray 整形数组原子类
    + AtomicLongArray 长整形数组原子类
    + AtomicReferenceArray 引用类型数组原子类

+ 引用类型
    + AtomicReference 引用类型原子类
    + AtomicStampedReference 原子更新引用类型里的字段原子类
    + AtomicMarkableReference 原子更新带有标记位的引用类型

+ 对象的属性修改类型
    + AtomicIntegerFieldUpdater 
    + AtomicLongFielfUpdater
    + AtomicStampedReference 

## 4.2 使用与原理
+ 以AtomicInteger为例，其方法如下：


        public final int get() //获取当前的值
        public final int getAndSet(int newValue)//获取当前的值，并设置新的值
        public final int getAndIncrement()//获取当前的值，并自增
        public final int getAndDecrement() //获取当前的值，并自减
        public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
        boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
        public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
    
+ AtomicInteger实现原理
    + 使用CAS以及volative来保证原子操作，从而避免synchronized的高开销，执行效率大为提升
    + CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值
    + UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset
    + 另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。





        // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
        private static final Unsafe unsafe = Unsafe.getUnsafe();
        private static final long valueOffset;

        static {
            try {
                valueOffset = unsafe.objectFieldOffset
                    (AtomicInteger.class.getDeclaredField("value"));
            } catch (Exception ex) { throw new Error(ex); }
        }

        private volatile int value;
# Reference
1. https://gitee.com/SnailClimb/JavaGuide/blob/master/docs/java/Multithread/synchronized.md
2. https://blog.csdn.net/zqz_zqz/article/details/70233767
3. https://www.baeldung.com/java-threadpooltaskexecutor-core-vs-max-poolsize