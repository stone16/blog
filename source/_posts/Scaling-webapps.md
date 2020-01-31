---
title: Scaling webapps
date: 2020-01-30 23:20:10
categories: Web
tags:
    - Web
    - Scalibility
top:
---
# 1. How scaling works 

## 1.1 Vertical scaling 
Run same things on a more powerful computer

## 1.2 Horizontal scaling 
Means run many processes in parallel. 

Nowadays, mostly we use horizontal scaling, since every computer internally have multi processors, we could do parallel programming to make whole thing work faster by nature.

# 2. Scaling process 

## 2.1 Initialization: single server and database

![fig1.png](https://i.loli.net/2020/01/31/klA2HgITCiwtRPc.png)

## 2.2 Reverse proxy 

![fig2.png](https://i.loli.net/2020/01/31/CgBvyKAW938H1Un.png)

+ Check if guests are allowed to enter 
+ A proxy is a process that receives and forwards requests 
+ Reverse means reqeust comes from teh internet and needs to be routed to our server 


Reverse proxy does following tasks: 
+ health check
    + make sure actual server is still up and running 
+ routing 
    + forward a request to the right endpoint 
+ authentication 
    + make sure that a user is actually permitted to access the server 
+ firewall
    + ensure users only have access to the parts they are allowed to use 


##  2.3 Load balancer 

![fig3.png](https://i.loli.net/2020/01/31/YR6kcGwTFC8JQf3.png)

Most reverse proxy can also act as load balancers. Load balancer's job is to split incoming requests among those servers

## 2.4 Grow database 
![fig4.png](https://i.loli.net/2020/01/31/LVqBUhFscRAgt9w.png)

+ Scaling database needs to deal with consistency 
+ Master - slace setup or write with read-replicas 
    + one part is exclusively responsible for receiving data and storing it 
    + all other parts are responsible for retrieving the stored data 


## 2.5 Microservices 
![fig5.png](https://i.loli.net/2020/01/31/Qj5ibmHAzLSWeqJ.png)

+ why we need microservices? 
    + different part use server to different extends 
    + development - might have more overlaps in singe service 

+ break server down into functional units and deploy them as individual, inter connected mini servers 

## 2.6 Caching & Content Delivery Networks 
![fig6.png](https://i.loli.net/2020/01/31/NRO6oguPe8JHMWb.png)

+ Large portion of our web app consists of static assets - almost never change 
    + use cache to speed it up 

## 2.7 Message Queues

![fig7.png](https://i.loli.net/2020/01/31/yXA4rKHbvw7cfuF.png)

+ deal with the waiting E.G fackbook deal with uploaded images 
    + store the raw, unprocessed image 
    + confirm the upload to users 
    + add to queue to process in the near future (async)
+ benefits 
    + decouple tasks and processors 
    + scale on demand 

## 2.8 sharding 

![fig8.png](https://i.loli.net/2020/01/31/dXanh3b217s4Miq.png)

+ A technique of parallelizing an application's stacks by separating them into multiple units, each responsible for a certain key or namespace 
+ shard based on location, use frequency and so on


## 2.9 load balancer 

![fig9.png](https://i.loli.net/2020/01/31/4FblrsEOeP6982U.png)

Load balancer has hard limit of how many requests they can handle

+ DNS
    + free load balancers 
    + registry allows you to specify multiple IPs per domain name, each leading to a different load balancer 

# reference 

https://arcentry.com/blog/scaling-webapps-for-newbs-and-non-techies/

