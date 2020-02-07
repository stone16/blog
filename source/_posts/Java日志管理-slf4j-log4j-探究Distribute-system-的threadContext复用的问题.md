---
title: Java日志管理 - slf4j + log4j - 探究Distribute system 的threadContext复用的问题
date: 2020-02-06 19:48:00
categories: BackEnd
tags:
    - Java
    - Log
top:
---
# 1. 常用日志系统介绍

+ 日志门面
    + jcl - jakarta common logging  
        + 日志接口，支持运行时动态加载日志组件的实现 
    + slf4j
        + 日志接口
        + 在log层和代码层之间起到门面作用，对于用户来说只要使用SLF4J的接口，就可以隐藏日志的具体实现，其提供的核心API是一些接口和一个LoggerFactory的工厂类，用户只需要按照其提供的日志接口进行使用，最终日志的格式，记录级别，输出方式等都可以通过具体日志系统的配置来实现
+ 日志框架
    + jul - java.util.logging 
    + log4j
    + log4j2
    + logback
        + slf4j的实现 
        + 是log4j的升级版

# 2. 日志级别

+ severe
+ warning 
+ info
+ config
+ fine 
+ finer 

# 3. 常见使用方式

一般工程上会选择一个日志门面，加日志框架，常用的门面基本上还是SLF4J比较多，框架的选择很多，主要是需要注意工程上分布式系统的多线程方面的支持。

# 4. Log4j2 ThreadContext复用的问题

ThreadContext出现的背景主要是因为现代系统基本上都需要都是和多个client打交道，在一个典型的多线程的应用系统当中，不同的线程就需要和不同的client打交道.为了能够分辨出同一个client发出的不同log请求，我们就需要给每个请求一个id，来做标记。

Log4j2使用Thread Context Map和Thread Context Stack来进行标记。常用Map，因为键值对更容易进行添加和处理

使用Stack: 

    ThreadContext.push(UUID.randomUUID().toString()); // Add the fishtag;
     
    logger.debug("Message 1");
    .
    .
    .
    logger.debug("Message 2");
    .
    .
    ThreadContext.pop();
    
使用Map: 

    ThreadContext.put("id", UUID.randomUUID().toString()); // Add the fishtag;
    ThreadContext.put("ipAddress", request.getRemoteAddr());
    ThreadContext.put("loginId", session.getAttribute("loginId"));
    ThreadContext.put("hostName", request.getServerName());
    .
    logger.debug("Message 1");
    .
    .
    logger.debug("Message 2");
    .
    .
    ThreadContext.clear();
    
问题来了，如果我们使用Executor.execute(new CachedThreadPool())的话，因为线程的复用，（所有使用完的线程会归还到线程池当中），那么ThreadContext里面的东西必须要进行手动清除，才能防止原先记录的这个线程的东西被复用掉。比如一般会记录的[Client][API]此类信息。

这个时候需要使用getContext()和cloneStack()来使得子线程获得父线程的信息。

    ThreadContext.clearAll();
    ThreadContext.putAll(preSavedThreadContextMap);
    ThreadContext.setStack(preSavedThreadContextStack);