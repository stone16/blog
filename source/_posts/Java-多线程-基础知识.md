---
title: Java 多线程
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