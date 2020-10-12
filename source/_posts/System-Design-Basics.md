---
title: System Design Basics
date: 2020-10-11 21:54:08
categories: SystemDesign
tags:
top:
---
# 1. Key Characteristics of Distributed Systems

## 1.1 Scalability 

+ Capability of a system, process, or a network to grow and manage increased demand. Achieve the scaling without performance loss 
+ Why need to scale?
    + Increased data volume 
    + Increased amount of work 
        + number of transactions 

+ Performance curve 
    + Usually performance of a system would decline with the system size due to the management or environment cost 
        + network speed come to be slower because machines tend to be far apart from one another 
        + some tasks may not be distributed, either because of their inherent atomic nature or because of some flaw in the system design 


+ Horizontal vs Vertical Scaling 
    + Horizontal Scaling 
        + Scale by adding more servers into your pool of resources 
        + Easier to scale dynamically by adding more machines into the existing pool 

    + Vertical scaling 
        + Scale by adding more power to an existing server 
            + CPU
            + RAM
            + Storage 

        + limited to the capacity of a single server, and scaling beyond that capacity often **involves downtime** and comes with an upper limit 


## 1.2 Reliability 

+ Probability a system will fail in a given period
+ A distributed system is considered reliable if it keeps delivering its services even when one or more software or hardware components fail 
+ A reliable distributed system achieves this through redundancy of both the software components and data 
    + Eliminate every single point of failure


## 1.3 Availability 

+ The time a system remains operational to perform its required function in s specific period 
+ Percentage of time that a system, service, or a machine remains operational under normal conditions 
+ Reliability  is availability over time considering the full range of possible real world conditions that can occur
+ If a system is reliable, it is available. However, if it is available, it is not necessarily reliable. In other words, high reliability contributes to high availability, but it is possible to achieve a high availability even with an unreliable product by minimizing repair time and ensuring that spares are always available when they are needed

## 1.4 Efficiency 

+ Standard measures of its efficiency 
    + Response time/ latency 
        + denotes the delay to obtain the first item 

    + Throughput / bandwidth
        + number of items delivered in a given time unit 

+ two measures above correspond to the following unit costs 
    + number of messages globally sent by the nodes of the system regardless of the message size 
    + size of messages representing the volume of data exchanges 


## 1.5 Serviceability / Manageability 

+ How easy it is to operate and maintain 
+ Simplicity and speed with which a system can be repared or maintained 
+ Ease of diagnosing and understanding problems when they occur, ease of making updates or modifications, 
