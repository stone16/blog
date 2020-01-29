---
title: 'Developing on AWS Note.9 Step Functions, ElasticCache'
date: 2020-01-28 23:18:45
categories: Cloud
tags:
    - AWS
    - Step Functions
    - ElasticCache
top:
---
# 1. Understanding the need for step functions 

+ Step functions make it easy to coordinate components of distributed applications and microservices by using visual workflows. 
+ Microservices are processes that communicate with each other over a network to complete a larger goal. 
+ Applications built as a collections of microservices are more resilient and easier to scale. 

+ Have cases that we begin to have more functions and they continue to grow
+ Need a mechanism to scale out, easily handle errors and timeouts, easily build and operate

# 2. Intro to AWS Step Functions

+ define a workflow called a state machine made up of states
+ each order is an execution through this state machine 
+ each execution starts with an input and the states transform it
+ step functions keep track of the state of each execution 
+ actually, it's a web service that enables you to coordinate the components of distributed applications and microservices using visual workflows.
+ provides a reliable way to coordinate components and step through the functions of your application. 
+ automatically triggers and tracks each step, and retries when there are errors
+ lifecycle
    + define workflow as a series of steps and transitions between each step, also known as a state machine
    + step functions ingests your JSON template and turns it into a real-time graphical view , help you make sense of your state machine's current state
+ benefits 
    + productivity
        + build applications quickly  
    + agility
        + scale and recover reliably 
    + resilience 
        + evolve applications easily 
+ Terminology
    + state machine - workflow template
        + an object that has a set number of operating conditions that depend on its previous condition to determine output 
        + AWS step functions allows you to create and automate state machines within the AWS env
            + with the use of a JSON-based Amazon State Language
            + a collection states, that can to work (task states), determine which states to transition to next (Choicestates), stop an execution with an error
        + state common features
            + each state must have a type field indicating what type of state it is
            + each state can have an optinal comment field to hold a human readable comment about, or description of, the state
            + Each state (except a Succeed or Fail state) requires a Next field or, alternatively, can become a terminal state by specifying an End field.
        + state types
            + task 
            + choise - adds branching logic
            + parallel 
            + wait
            + fail
            + succeed 
            + pass
    + execution - specific workflow based on template
    + task - lambda function or activity 
        + An activity consists of program code or a task that waits for **an operator to perform** an action or to provide input. You can host activities on Amazon EC2, on Amazon ECS, or even on mobile devices. Activities poll Step Functions using the **GetActivityTask** and **SendTaskSuccess**, **SendTaskFailure**, and **SendTaskHeartbeat** API actions. Activities represent workers (processes or threads), implemented and hosted by you, that perform a specific task.
        + A Lambda function is a cloud-native task that runs on AWS Lambda. You can write Lambda functions in a variety of programming languages, using the AWS Management Console or by uploading code to Lambda. Lambda functions execute a function using AWS Lambda. To specify a Lambda function, use the ARN of the Lambda function in the Resource field 
    + activity - handle for external compute
    + task token - ID for instance of activity 
    + heartbeat - ping from task indicating that it is still running 
    + failure 
    + success 

# 3. Caching for scalibility 

## 3.1 Caching Overview

+ Benefits
    + Provides high throughput, low latency access to commonly accessed application data, **by storing the data in memory**
    + imrpove the speed 
    + reduce the response latency 
    + the following types of information or applications can benefit from caching:
        + results of database queries
        + results of intensive calculations
        + results of remote API calls 


+ when to consider caching your data
    + data that requires a slow and expensive query to acquire 
    + relatively static and frequently accessed data 
    + information that can afford to be stale for some time
    + data should be relatively static and frequently accessed
    + Cache data should always be considered and treated as stale


## 3.2 Caching Strategy

# 4. Amazon ElastiCache
    
+ a webservice that makes it easy to deploy, operate and scale an in-memory cache in the cloud
+ ElastiCache improves the performance of web applications by allowing you to retrieve information from fast, managed, in-memory caches, instead of relying entirely on slower disk-based databases 
+ ElastiCache supports
    + Memcached
    + Redis

## 4.1 Memcached vs Redis

+ Memcached
    + multithreading 
    + low maintenance 
    + easy horizontal scalability with auto discovery 
    + single AZ 
    + lack persistence
        + if you terminate node or scale it down, you lose the data stored in that cache memory  
+ Redis
    + single thread 
    + support for data structures 
        + strings
        + hashes
        + lists
        + sets
        + sorted sets with range queries
        + bitmaps
        + hypolog
        + geospatial indexes with radius queries
    + persistence 
    + atomic operations 
    + pub/ sub messaging
    + read replicas/ failover
    + cluster mode/ sharded clusters
    + multiple AZ 

## 4.2 Terminology 

+ node
    + smallest building block of an ElastiCache deployment 
+ cluster
    + a logical grouping of one or more nodes
+ replication group
    + a collection of Redis clusters
    + with one primary read-write cluster and up to five secondary, read only clusters, which are called read replicas. 
    + each read replica maintains a copy of the data from the primary cluster
    + asynchronous replication mechanisms are used to keep the read-replicas synchronized with the primary cluster 
    + applications can **read from any cluster in the replication group** 
    + applications can write only to the primary cluster
    + read replicas enhance scalability and guard against data loss 

## 4.3 Cache hit & Cache Miss Scenarios 

+ cache hit occurs when the cache contains the information required
+ cache miss occurs when the cache does not contain the information requested
+ ElastiCache caches data as key-value pairs.
+ An application can retrieve a value corresponding to a specific key. 
+ An application can store an item in cache by specifying a key, value, and an expiration time(TTL). 


## 4.4 Cache strategies

+ Lazy loading
    + Whenever your application requests data, it first makes the request to the ElastiCache cache. 
    + If the data exists in the cache and is current, ElastiCache returns the data to your application. 
    + If the data does not exist in the cache, or the data in the cache has expired, your application requests the data from your data store which returns the data to your application. 
    + Your application then writes the data received from the store to the cache so it can be more quickly retrieved next time it is requested.
    + Lazy Loading is a caching strategy that loads data into the cache only when necessary. 
    + Avoid filling up the cache with unnecessary data
    + advantages
        + Only requested data is cached. Since most data is never requested, lazy loading avoids filling up the cache with data that isn't requested.
        + Node failures are not fatal. 
        + When a node fails and is replaced by a new, empty node the application continues to function, though with increased latency. As requests are made to the new node each cache miss results in a query of the database and adding the data copy to the cache so that subsequent requests are retrieved from the cache.
    + disadvantages
        + a cache miss penalty, each cache miss results in 3 trips
            + initial request for data from the cache
            + query of the database for the data
            + write the data to the cache 
        + may receive stale data because another application may have updated the data in the database behind the scenes. 
+ write through 
    + this strategy adds data or updates data in the cache whenever data is written to the database 
    + advantages
        + data in the cache is never stale, always current
    + disadvantages
        + write penalty, involve two trips
            + a write to cache, and a write to the database
            + missing data: When a new node is created to scale up or to replace a failed node, the node does not contain all data. Data continues to be missing until it is added or updated in the database. In this scenario, you might choose to use a lazy caching approach to repopulate the cache
            + Unused data: Since most data is never read, there can be a lot of data in the cluster that is never read.
            + Cache churn: The cache may be updated often if certain records are updated repeatedly.
