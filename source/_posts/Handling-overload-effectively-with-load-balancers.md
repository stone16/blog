---
title: Handling overload effectively with load balancers
date: 2020-02-04 09:13:39
categories: BackEnd
tags:
    - Load Balancers
top:
---
# 1. Fail Smarter 

+ Average latency has a spike, availability down
+ reasons 
    + dependencies 
    + cascading effect 
    + client change   peak 

# 2. Load balancers 

+ clients - load balancers - servers 
+ hardware devices 
+ multi tenant - use VIP - for efficiency 
+ used for (features)
    + scaling 
    + even traffic distribution 
    + overload protection
+ how
    + how to pick a server 
        + random dice 
        + round robin 
        + least conns 
    + desired algorithm - we use least conns 
        + simple 
        + reliable 
        + even 
    + max conns? 
    + how to deal with overload? 
        +  attributes 
            + cheap 
            + local 
            + buffering - having capacity soon 
            + priority 
        + reject requests  - spillover - choose! 
            + close the TCP connection 
            + no buffer 
        + hang on, wait in a queue - surge queue 
            + may mkes it take longer
+ some default we set - maxConns - perhost setting 
    + little's law 
        +  arrival rate * time shopping = people in the store 
    + see load balancers how many services we have 
    + fleet-wide concurrent requests / host count = **avg conns**


+ metrics matter 
    + latency netwrok latency + 25% 
        + client side 
        + server side 
    + request rate 
+ coral server 
    + concurrent requests - outstanding request - in one host 
+ 33% overhead room - dependency failures 


# 3. Actual behave 

+ increase load to see average latency 
+ run actual test 
+ generate graph with outstanding requests 
    + see the cross of client timeout and p99 

# 4. Abnormal cases 

+ dependency latency, timeout 
+ network  packet loss 


+ let server decide what's the maxCon should be 
+ coral has
    + connection 
    + worker  work thread 
+ classify requests 
    + importance 
        + droppable 
    + priority 
        + order  