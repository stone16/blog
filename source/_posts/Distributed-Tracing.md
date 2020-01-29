---
title: Distributed Tracing
date: 2020-01-28 23:39:40
categories: Cloud
tags:
    - Distributed Tracing 
    - System Design
top:
---

# 1. Why we need it? 

Short answer - due to disturbuted applications. 

## 1.1 Monolithis software 
Monolithis software is build upon a large and sprawling legacy code base that is often so tightly coupled that any changes in one small section often result in breaking one or several features that depend on it. In such app, high possibly it'll break and we need to use tech - tracing to **follw the course of a request or system event** from its source to its ultimate destination. 

In this way, each trace comes to be a narrative that tells the request's story as it travels through system. 

## 2.2 Distributed System 

Use distributed tracing to profile and monitor microservice-based apps/ architectures, locate failures, and improve performance. 

# 2. Key Concepts 

In general, distributed tracing start with a single request - the entity or event being traced. As the request makes its journey, it **generates traces that record complete processing operations** performed on it by entities within a distributed system/ network infrastructure. 

Each trace is assigned with its own unique ID and passes through a segment that indicates a given activity that a host system performs on the request. Every segments represents a single step within the reqeust's path and has a name, unique ID, and timestamp. A span(segment) can also carry additional metadata. 

The idea is -- specific request inflexion points mush be identified within a system and instrumented. All of the trace data mush be coordinated and collated to provide a meaningflow view of a request. 

Challenge would be processing the volume of the data generated from increasingly large scale systems. 

# 3. Implementation 

Google created Dapper in the past as a middleware that supports using different language within the system. As said, the value of tracing is only realised through: 

+ ubiquitous deployment, and no parts of the system under observation are not instrumented 
+ continuous monitoring 
    + system mush be monitoring constantly 


# 4. Why we need distributed tracing

Greg Linden commented in 2006 that experiments ran by Amazon.com demonstrated a [significant drop in revenue](http://glinden.blogspot.com/2006/11/marissa-mayer-at-web-20.html) was experienced when 100ms delay to page load was added. Although understanding the flow of a web request through a system can be challenging, there can be significant commercial gains if performance bottlenecks are identified and eliminated.


# Reference
1. https://epsagon.com/blog/introduction-to-distributed-tracing/
2. https://www.infoq.com/articles/distributed-tracing-microservices/
3. https://www.javacodegeeks.com/microservices-distributed-tracing.html
4. https://ai.google/research/pubs/pub36356 