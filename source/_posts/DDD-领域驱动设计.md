---
title: DDD-领域驱动设计
date: 2020-02-02 08:04:32
categories: SystemDesign 
tags:
    - DDD
top:
---

# 1. Intro 

## 1.1 什么是领域驱动设计
、】=op90q`

> DDD是面向对象的一套方言，提供了一种基于业务领域的对象划分和分类方法，其最大的价值就在于对于软件开发过程全生命周期使用语言的统一。

## 1.2 常用名词

+ Controller 
+ Service 
+ Repository 
+ Entity 


## 1.3 为什么使用DDD? 
我们在使用面向对象语言来解决实际问题的时候，知道所有东西都是对象，但我们并没有明确的关于对象应当如何组织和划分的规范。

作为技术人员，我们习惯于从技术角度来出发进行思考，出现了用**技术语言**定义对象的习惯，例如DAO(Data Access Objects), DTO(Data Transfer Object)这类对象及其体现出来的划分方式。

但是我们日常工作当中很多时间需要做大量的业务语言和技术语言的相互翻译。DDD - Domain Driven Design 就是想解决这样的相互翻译的问题，希望能通过一套面向对象的分类方法，从领域触发，实现软件开发过程当中各个角色和环境的统一语言(Ubiquitous Language).例如使用Repository（仓库）替代数据访问对象（DAO），就更能让业务和技术同时理解这个对象的作用了。


DDD分为战术和战略两部分，恰好可以用在微服务的划分当中。我们需要利用DDD的战略部分，划分问题域，来合理对微服务进行划分。

# 2. 事件风暴 EventStorming  

> 是一套独立的通过协作基于事件还原系统全貌，从而快速分析复杂业务领域，完成领域建模的方法。

# Reference
1. https://insights.thoughtworks.cn/ddd-eventstorming-zhongtai/#utm_source=rss&utm_medium=rss
2. https://www.eventstorming.com/#services