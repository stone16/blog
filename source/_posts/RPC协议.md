---
title: RPC协议
date: 2020-01-31 20:59:08
categories: Web
tags:
    - Web
    - RPC
top:
---

# 1. 什么是远程过程调用

RPC是指计算机A上的进程，调用另外一台计算机B上的进程，其中A上的调用进程被挂起，而B上的被调用进程开始执行，当值返回给A时，A进程继续执行。

注意这中间，AB都经历了内核态和用户态之间的转变。

整个流程如图所示：

![fig1.jpg](https://i.loli.net/2020/02/01/tqlp2wST4KBcgyn.jpg)


# 2. 痛点

那远程过程调用到底解决了什么问题呢？

由于各服务部署在不同机器上，要想再服务间进行远程调用免不了网络通信过程，服务消费方每调用一个服务都要写大量和网络通信相关的代码，RPC框架实际上完成了对网络通信的细节的封装，让消费方能够像调用本地服务一样调用远程服务。

# 3. RPC框架入门

RPC框架中主要有三个角色，
+ Provider 服务提供方
+ Consumer 服务消费方
+ Registry 服务注册中心

使用服务注册中心的原因是，在SOA框架中，往往Provider和Consumer的数量不唯一，通过注册中心注册服务，可以做负载均衡。

![fig2.jpg](https://i.loli.net/2020/02/01/mYIbBUGoSEPTvu5.jpg)

![fig3.jpg](https://i.loli.net/2020/02/01/HqPaDVrWStugIf6.jpg)


服务提供者启动后主动向注册中心注册机器ip, port以及提供的服务列表；服务消费者启动时向注册中心获取服务提供方地址列表，可实现软负载均衡和Failover. 
RPC框架当中需要使用很多技术，以下列出来主要的一部分： (do match internal amazon c***** framework!)

## 3.1 动态代理

## 3.2 序列化

为了能在网络上传输和接收Java对象，需要进行序列化和反序列化的操作

## 3.3 NIO
基于Netty这一IO通信框架

[Netty4详解](https://blog.csdn.net/suifeng3051/article/details/23348587)

## 3.4 服务注册中心

Redis/ ZooKeeper/ Consul/ Etcd

# 4. RPC vs REST

是理念的不同，REST是一种设计风格，REST的URL主体是资源，是名词，而且也仅支持HTTP协议，规定了使用HTTP Method来表达本次要做的动作，类型个位数... 这些动作表达了对资源仅有的几种转换方式。

而RPC的思想，是把本地函数映射到API，也就是说一个API对应的是一个函数，本地有什么函数，远程也可以调用这个函数；是建立在采用的协议之上的。

可以参考[WEB开发中，使用JSON-RPC好，还是RESTful API好？](https://www.zhihu.com/question/28570307)获取更多细节。
