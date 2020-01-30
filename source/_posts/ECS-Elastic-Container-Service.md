---
title: ECS - Elastic Container Service
date: 2020-01-29 20:36:54
categories: Cloud
tags:
    - AWS
    - ECS
top:
---

# 1. Intro

+ Highly scalable, fast, container management service that makes it easy to run, stop and maange docker containers on a cluster. 


![fig1.png](https://i.loli.net/2020/01/30/bgWNQv1M27fEza3.png)
# 2. Concepts
+ Containers
    + Application components must be architected to run in containers 
    + a standardized unit of development 
    + contain everything application needs
        + code 
        + runtime 
        + system tools 
        + libraries 
+ Images
    + containers are created here
    + A read only templates
+ Registry
    + used to store images  
+ Task 
    + text file in JSON 
    + describes 1-10 containers to form your application 
+ Cluster 
    + logical group of resources  
+ ECS Task scheduler
    + responsible for placing tasks within cluster  
# 3. Features

+ Regional service 


# 4. Steps for starting with Fargate

+ Create a task definition 
+ Configure the service 
    + a service launches and maintains a specified number of copies of the task definition in cluster   
+ configure the cluster 