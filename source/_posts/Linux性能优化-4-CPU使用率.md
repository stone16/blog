---
title: Linux性能优化(4)-CPU使用率
date: 2020-01-30 20:52:23
categories: Linux
tags:
    - Linux
    - CPU Utilization
top:
---

衡量CPU的方式，可以使用：
+ 平均负载
+ CPU上下文切换
+ CPU使用率

# 1. CPU使用率
## 1.1 概念
Linux作为一个多任务操作系统，将每个CPU的时间划分为很短的时间片，再通过调度器轮流分配给各个人物使用，因此造成多任务同时运行的错觉。

为了维护CPU时间，Linux通过事先定义的节拍率，触发时间中断，并使用全局变量Jiffies记录了开机以来的节拍数，每发生一次时间中断，Jiffies的值就加1.可以通过查询`/boot/config`内核选项来查看其配置，一般来说设置为100，250， 1000等数值。比如对于100来说，就是每秒触发250次时间中断。

Linux通过`/proc`虚拟文件系统，向用户空间提供系统内部状态的信息，而`/proc/stat`提供的就是系统的CPU和任务统计信息

+ 使用man proc 查询, 都是和CPU使用率相关的重要指标
    + user: 用户态CPU时间
    + nice: 低优先级用户态时间
    + system: 内核态CPU时间
    + idle: 空闲时间
    + iowait: 等待I/O的CPU时间
    + irq: 处理硬中断的CPU时间
    + softirq: 处理软中断的CPU时间
    + steal: 运行在虚拟机当中，被其他虚拟机占用的CPU时间
    + guest: 运行虚拟机的CPU时间
    + guest_nice: 以低优先级运行虚拟机的时间

![fig1.png](https://i.loli.net/2020/01/31/vbKiQhfYDls4XrH.png)

## 1.2 如何查看

+ top
    + 显示系统总体的CPU和内存使用的情况，以及各个进程的资源使用情况 
+ ps
    + 显示每个进程的资源使用情况 
+ pidstat
    + 分析每个进程的CPU使用率 

## 1.3 如何分析

+ 使用Perf
    + perf top
        + 显示占用CPU时钟最多的函数或者指令 
    + perf record
        + 可以保存数据 
    + perf report