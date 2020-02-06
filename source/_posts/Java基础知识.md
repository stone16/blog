---
title: Java基础知识
date: 2020-02-05 19:05:08
categories: BackEnd
tags:
    - Java
top:
---
给自己搭个脚手架，建个基础知识的小字典，查遗用(持续更新)：

# 1.Java平台
## 1.1 Java解释执行？

Java源代码，通过javac编译成字节码，运行时通过JVM内嵌的解释器将字节码转换为机器码。但是大部分JVM都提供了JIT(just in time),即动态编译器，它能够在运行时将热点代码编成机器码，这种情况下热点代码就属于编译执行。

## 1.2 Java类加载机制

类加载大致过程：加载-验证-链接-初始化

## 1.3 Java反射机制

## 1.4 面向对象编程SOLID原则

1. Single Responsibility 
2. Open for extension, close for modification 
3. Liskov Substitution 
4. Interface Segragation 
5. Dependency Injection 


E.G

    public class VIPCenter {
      void serviceVIP(T extend User user>) {
         if (user instanceof SlumDogVIP) {
            // 穷 X VIP，活动抢的那种
            // do somthing
          } else if(user instanceof RealVIP) {
            // do somthing
          }
          // ...
      }
增加其扩展性：

    public class VIPCenter {
       private Map<User.TYPE, ServiceProvider> providers;
       void serviceVIP(T extend User user） {
          providers.get(user.getType()).service(user);
       }
     }
     interface ServiceProvider{
       void service(T extend User user) ;
     }
     class SlumDogVIPServiceProvider implements ServiceProvider{
       void service(T extend User user){
         // do somthing
       }
     }
     class RealVIPServiceProvider implements ServiceProvider{
       void service(T extend User user) {
         // do something
       }
     } 

在另一篇博文里，有对SOLID的详细描述，详情见[面向对象设计原则](https://www.llchen60.com/2018/11/12/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99/)

## 1.5 类加载过程

load-link-initialize

+ load 

首先是加载阶段（Loading），它是 Java 将字节码数据从不同的数据源读取到 JVM 中，并映射为 JVM 认可的数据结构（Class 对象），这里的数据源可能是各种各样的形态，如 jar 文件、class 文件，甚至是网络数据源等；如果输入数据不是 ClassFile 的结构，则会抛出 ClassFormatError。

+ link 

第二阶段是链接（Linking），这是核心的步骤，简单说是把原始的类定义信息平滑地转化入 JVM 运行的过程中。这里可进一步细分为三个步骤：

1. 验证， JVM需要验证字节信息是符合Java虚拟机规范的，否则会被认为是Verify Error
2. 准备 创建类或接口中的静态变量，并初始化静态变量的初始值，侧重点在分配所需要的内存空间，不会去执行更进一步的JVM指令
3. 解析。将常量池中的符号引用替换为直接引用。


再来谈谈双亲委派模型，简单说就是当类加载器（Class-Loader）试图加载某个类型的时候，除非父加载器找不到相应类型，否则尽量将这个任务代理给当前加载器的父加载器去做。使用委派模型的目的是避免重复加载 Java 类型。

# 2. Java语言特性
## 2.1 泛型

## 2.2 Lambda

## 2.3 Exception/ Error

### 2.3.1 定义
> Exception: 程序正常运行中可以预料到的意外情况，可能并且应该被捕获，进行相应处理

> Error: 在正常情况下，不应该出现的情况。绝大部分的Error都会导致程序比如JVM自身处于非正常的、不可恢复的状态

Throwable分类图！！！！！

异常之所以很强大，在调试方面，在于其回答了以下三个问题：

1. 什么出了错？    异常类型
2. 在哪出了错？    异常堆栈跟踪位置
3. 为什么出错？    异常信息

### 2.3.2 Tips

1. 不捕获通用异常，写自己的Exception，方便Debug
2. 不要生吞异常，不知道怎么处理了可以继续向外层抛出
3. 提早抛出，延迟捕获！ 
4. 把异常处理的责任往调用链的上游传递的方法就是在方法的throws子句声明异常。[(如何优雅的处理异常？)](https://www.zhihu.com/question/28254987)

### 2.3.3 性能角度分析

1. try-catch代码段会产生额外的性能开销，往往会影响JVM对代码进行优化，因此应该仅捕获有必要的代码，尽量不要使用一个大的try包住整段代码
2. Java每实例化一个Exception，都会对**当时的栈进行快照**，这是个比较重的操作。

## 2.4 引用

Java中除了原始数据类型的变量其他所有都是引用类型，指向不同的对象。理解引用，以理解Java对象的生命周期和JVM内部的相关机制。

不同的引用类型，主要体现在对象的不同的可达性状态和对垃圾收集的影响。

### 2.4.1 强引用
Strong Reference, 即普通对象引用。只要还有强引用指向一个对象，那么垃圾收集器就不会碰。

### 2.4.2 弱引用
不能使对象豁免垃圾收集，提供一种访问在弱引用装天下对象的途径

使用weak reference类来实现的

### 2.4.3 软引用
Soft Reference. 相对强引用弱化一些的引用，可以让对象豁免一些垃圾收集，只有当JVM认为内存不足时，才会去试图回收软引用指向的对象。

使用soft reference类实现的

### 2.4.4 幻象引用
虚引用，提供一种确保对象被finalize以后，做某些事情的机制。

通过PhantomReference类来实现的

## 2.5 String vs. StringBuffer vs. StringBuilder
+ 字符串缓存，放在对重，后来放在metaSpace当中，可以通过JVM调参来修改大小

    -XX:+PrintStringTableStatistics
    -XX:StringTableSize=N

+ String 压缩

从使用Char到使用Byte + 标志编码位。紧凑字符串带来了很大优势，更小的内存占用，更快的操作速度。
### 2.5.1 String
Immutable类，声明为final class,所有属性也都是final的。

java引入了字符创常量池，创建一个字符串，首先看常量池里面有没有值相同的字符串，如果有直接到池里拿对应的对象引用，如果没有就创建新的，并放到池里去。

    String str1 = "123";// 放入常量池
    String str2 = new String("123"); // 不放入常量池

### 2.5.2 StringBuffer
为了解决String拼接产生太多中间对象的问题，本质上是一个线程安全的可修改字符序列。保证了线程安全，但是也带来了额外的性能开销。

线程安全是通过在各种修改数据的方法上加`synchronized`关键字来实现的。

底层使用char/ byte数组，继承了AbstractStringBuilder

### 2.5.3 StringBuilder 
与StringBuffer类似，去掉了线程安全的部分，有效减少了开销

底层使用char/ byte数组，继承了AbstractStringBuilder。

## 2.6 Abstract and Interface 
### 2.6.1 接口
接口是对行为的抽象，是抽象方法的集合，利用接口可以达到API定义和实现分离的目的。接口不能实例化，不能包含任何非常量成员，任何field都是隐含着public static final的意义的。同时，没有非静态方法的实现。要么是静态方法，要么是抽象方法。

接口可以多继承！类只可以单继承

### 2.6.2 抽象类

不能实例化的类，用abstract关键字来修饰，其目的主要是代码重用。抽象类大多用于抽取相关Java类的公用方法实现或者是共同成员变量，然后通过继承达到代码复用的目的。Java标准库中，比如Collection框架，很多通用部分就抽象成了抽象类来使用。



# 3. Java基础类库
## 3.1 集合
### 3.1.1 Vector vs. ArrayList vs.LinkedList 
三者都是集合框架中的List，根据位置有定位，添加或者删除的操作。Vector是Java早期提供的线程安全的动态数组。ArrayList动态数组，不是线程安全的，ArrayList扩容时会增加50%。LinkedList双向链表，不是线程安全的。

Vector, arrayList作为动态数组，非常适合随机访问的场合，插入删除元素性能会比较差，因为要移动后续的所有元素。LinkedList进行节点插入，删除会很高效，但是随机访问性能很差。

### 3.1.2 Hashtable vs. HashMap vs. TreeMap 

Hashtable是同步的，性能开销大

HashMap 非同步的，性能开销小，put，get操作能达到常数时间的性能

TreeMap是基于红黑树的一种提供顺序访问的Map，和Hashmap不同，它的get\put\remove之类的操作都是O(log(n))的时间复杂度。



## 3.2 IO/NIO
### 3.2.1 IO
java.io包，基于流模型实现，提供了我们最熟知的一些IO功能，比如File抽象、输入输出流等。交互方式是同步、阻塞的方式。也就是说，在读取输入流或者写入输出流时，在读写动作之前，线程会一直阻塞在那里，他们之间的调用是可靠的线性顺序。

### 3.2.2 NIO 
NIO框架，提供Channel, selector, Buffer等新的抽象，可以构建**多路复用，同步非阻塞IO程序**，同时提供了更接近操作系统底层的高性能数据操作方式。

+ Channel 
+ Buffer
+ Selector 




## 3.3 网络

## 3.4 并发
### 3.4.1 synchronized 
+ 概念

是java内部的同步机制，也称为intrinsic locking, 提供了互斥的语义和可见性，当一个线程已经获取当前锁时，其他试图获取的线程只能等待或者阻塞在那里了。

在 Java 5 以前，synchronized 是仅有的同步手段，在代码中， synchronized 可以用来修饰方法，也可以使用在特定的代码块儿上，本质上** synchronized 方法等同于把方法全部语句用 synchronized 块包起来**。

+ 底层实现

synchronized代码块是由一对`monitorenter/ moniterexit`指令来实现的，Monitor对象是同步的基本实现单元。

在 Java 6 之前，Monitor 的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。现代的（Oracle）JDK 中，JVM 对此进行了大刀阔斧地改进，提供了三种不同的 Monitor 实现，也就是常说的三种不同的锁：偏斜锁（Biased Locking）、轻量级锁和重量级锁，大大改进了其性能。

所谓锁的升级、降级，就是 JVM 优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。

没有竞争的时候，默认使用偏斜锁。JVM会利用CAS操作，在对象头上的Mark Word部分设置线程ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM就需要撤销偏斜锁，并切换到轻量锁的实现。


### 3.4.2 ReeantrantLock 


+ ReentrantLock 再入锁

再入锁通过代码直接调用lock()方法来获取，是表示当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功。这是多锁获取粒度上的一个区分，锁的持有是以线程为单位而不是基于调用次数了，java锁实现强调再入性是为了和pthread的行为进行区分。

再入锁可以设置公平性：

    ReentrantLock fairLock = new ReetrantLock(true);

这里所谓的公平性是指在竞争场景中，当公平性为真时，会倾向于将锁赋予等待时间最久的线程。公平性是减少线程“饥饿”（个别线程长期等待锁，但始终无法获取）情况发生的一个办法。

使用synchronized我们无法进行公平性的选择，其永远是不公平的，这也是主流操作系统线程调度的选择。通用场景中，公平性未必有想象中的那么重要，Java 默认的调度策略很少会导致 “饥饿”发生。与此同时，若要保证公平性则会引入额外开销，自然会导致一定的吞吐量下降。

### 3.4.3 死锁


## 3.5 安全

# 4. JVM基础概念和机制

## 4.1 类加载机制

## 4.2 常见的垃圾收集器



# Reference

1. [如何优雅的处理异常？](https://www.zhihu.com/question/28254987)