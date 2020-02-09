---
title: System.gc() and Runtime.gc()
date: 2020-02-08 22:10:53
categories: BackEnd
tags:
    - Java
top:
---
首先在Java中垃圾回收算法是首先遍历所有在堆中的非垃圾的对象，然后推断出那些一段时间内没有被访问的对象一定是垃圾了。call gc()方法不是强制垃圾回收发生的，相反的，它只是在建议JVM现在是不错的做垃圾回收的时间。

system.gc()是用来运行垃圾收集器的。call这个方法就意味着Java虚拟机正在努力去回收没有被使用的对象，使得他们现在占用的内存可以进行快速地再利用。整个垃圾回收在Java中是自动进行的。

system.gc()是个静态方法，但是手动调用它很有可能会让整个系统运行更慢的，一般为了加快整体的运行，会使用`-XX:+DisableExplicitGC`这条指令，这样子JVM就不会在你手动唤醒gc的时候直接call这个方法了。

runtime.gc()和system.gc()并没有什么区别，实质上system.gc()内部就call了runtime.gc()。 唯一的不同在于System.gc()是类的方法然而runtime.gc()是实例方法。


# Reference 

1. http://net-informations.com/java/cjava/gc.htm 
2. https://docs.oracle.com/javase/7/docs/api/java/lang/System.html 
3. https://www.geeksforgeeks.org/garbage-collection-java/ 