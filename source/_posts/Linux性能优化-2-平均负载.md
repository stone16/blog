---
title: Linux性能优化(2)-平均负载
date: 2020-01-30 20:48:55
categories: Linux
tags:
    - Linux
    - 平均负载
top:
---
# 1. 如何理解平均负载

    $uptime
    14:35:36 up 62 days,  2:51,  2 users,  load average: 0.02, 0.70, 1.06

当前时间，系统运行时间，正在登录用户数

后面的三个数字是过去1min, 5min, 15min的平均负载。

> 平均负载时指单位时间内，系统处于可运行状态和不可中断状态的平均进程数，也就是平均活跃进程数，它和CPU使用率没有直接关系。

> 可运行状态的进程：指正在使用CPU或者正在等待CPU的进程, 也就是我们用`ps`看到的处于Running/ Runnable状态的进程。

> 不可中断状态的进程：指正处于内核态关键流程中的进程，并且这些进程是不可以被打断的。比如等待硬件设备的I/O响应，即我们在`ps`命令下看到的D(Uninerruptible Sleep/ Disk Sleep)状态.

例如当一个进程向磁盘读写数据时，为了一致，在得到磁盘回复之前，是不能被其他进程或者中断打断的。否则容易出现磁盘数据与进程数据不一致的问题。

不可中断状态实际上是系统对进程和硬件设备的一种保护机制。

平均负载 -> 平均活跃进程数 -> 单位时间内的活跃进程数.最理想的情况就是每个CPU都刚好运行着一个进程，这样每一个CPU都能得到充分的利用。

# 2. 平均负载为多少时比较合理？

首先需要知道系统有几个CPU，可以通过top命令或者从文件/proc/cpuinfo中读取

    $grep 'model name' /proc/cpuinfo | wc -l

当平均负载高于CPU数量70%的时候，就应该分析和排查负载高的问题了。

# 3. 平均负载与CPU使用率

二者不一定会完全一致，因为平均负载指单位时间内处于可运行状态和不可中断状态的进程数，不仅包括了***正在使用CPU的进程，还包括等待CPU和等待I/O的进程***。

而CPU使用率，是根据单位时间内CPU的繁忙情况进行统计的，举例：

+ CPU密集进程

使用大量CPU使得平均负载升高，此时二者是一致的

+ IO密集型进程

等待I/O也会导致平均负载升高，但CPU使用率不一定会很高

# 4. 案例分析

## 4.1 使用工具

### 4.1.1 stress
Linux 系统压力测试工具，可以用作异常进程模拟平均负载升高的场景。

### 4.1.2 sysstat
包含了常用的多核CPU性能分析工具，用来实时查看进程的CPU、内存、I/O以及上下文切换等性能指标。

## 4.2 场景分析-CPU密集型进程

terminal 1: 

    $ stress --cpu 1 --timeout 600

terminal 2: 

    # -d 参数表示高亮显示变化的区域
    $ watch -d uptime
    ...,  load average: 1.00, 0.75, 0.39

terminal 3: run mpstat to see CPU usage 

    # -P ALL 表示监控所有 CPU，后面数字 5 表示间隔 5 秒后输出一组数据
    $ mpstat -P ALL 5
    Linux 4.15.0 (ubuntu) 09/22/18 _x86_64_ (2 CPU)
    13:30:06     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
    13:30:11     all   50.05    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   49.95
    13:30:11       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    13:30:11       1  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00


Use pidstat to see process usage 

    # 间隔 5 秒后输出一组数据
    $ pidstat -u 5 1
    13:37:07      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
    13:37:12        0      2962  100.00    0.00    0.00    0.00  100.00     1  stress


# 5. 总结

1. mpstat -P ALL 5
2. pidstat -u 5 1

+ -u : cpu usage
+ -r : memory usage
+ -d : I/O usage
+ -p : specific process id 

3. watch -d uptime 

watch可以帮你检测一个命令的运行结果，在Linux下，watch是周期性的执行下个程序，并全屏显示执行结果。你可以拿他来监测你想要的一切命令的结果变化，比如 tail 一个 log 文件，ls 监测某个文件的大小变化。

+ -n : interval
+ -d : highlight 
+ '' : command

e.g : `watch -n 1 -d 'pstree|grep http'`

4. uptime 


# 6. Reference 

1. [Linux CPU 实时监控mpstat命令详解](https://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858775.html)
2. [watch命令](http://www.cnblogs.com/peida/archive/2012/12/31/2840241.html)
3. [Linux运行进程实时监控pidstat命令详解](https://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858874.html)