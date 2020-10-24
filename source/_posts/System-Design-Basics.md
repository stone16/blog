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

# 2. Load Balancing 

## 2.1 What is Load Balancer? 

+ A critical component of any distributed system
    + help to spread the traffic across a cluster of servers to improve responsiveness and availability of applications/ websites/ databases 
    + also keep track of the status of all the resources while distributing requests 
        + if one server is not available to take new requests, not responding, has elevated error rate 
        + LB will stop sending traffic to such server 

    + sit between client and the server accepting incoming network and application traffic, distribute traffic across multiple backend servers using various algorithm
    + could prevents any one application server from becoming a single point of failure 

+ where to add LB?
    + between the user and the web server 
    + between web servers and an internal platform layer, like application servers or cache servers 
    + between internal platform layer and database 
## 2.2 Benefits 

+ from user side 
    + faster, uninterrupted service; their requests could be immediately passed on to a more readily available resource 

+ from service provider side 
    + experience less downtime and higher throughput 

+ long term benefits
    + smart load balancers provide benefits like predictive analytics that determine traffic bottlenecks before they happen 

## 2.3 Load Balancing Algorithms

+ How does the load balancer choose the backend server?
    + 1. Make sure servers could respond appropriately to requests 
        + routinely do health check 
            + regularly attempt to connect to backend servers to ensure that servers are listening 
    + 2. Use pre configured algorithm to select one from the set of healthy servers 
        + Least connection Method 
            + direct traffic to the server with fewest active connections 
            + useful when there are a large number of persistent client connections which are unevenly distributed between the servers

        + Least Response Time Method 
            + direct traffic to the server with the fewest active connections and the lowest average response time 

        + Least Bandwidth Method 
            + selects the server that is currently serving the least amount of traffic measured in megabits per second (Mbps)

        + Round Robin Method 
            + cycles through a list of servers and sends each new request to the next server 
            + most useful when the servers are of equal specification and there are not many persistent connections

        + weighted round robin 
            + desifned to better handle servers with different processing capacities 
            + each server is assigned a weight which indicates the processing capacity 

        + IP Hash 
            + A hash of the IP address of the client is calculated to redirect the request to a server 


## 2.4 Redundant Load Balancers 

+ Load Balancer can be a single point of failure 
    + thus we need a second load balancer, to form a cluster 
    + each LB monitors the health of the other
    + passive one could be switched to be active anytime since they keep monitoring same 

# 3. Caching 

Caching enable you to make vastly better use of the resources you already have as well as make otherwise unattainable product requirements feasible。 

It takes advantage of the locality of reference principle: recently requested data is likely to be requested again, could be used in almost every layer of computing 

## 3.1 Application Server Cache 

+ place a cache directly on a request layer node 
+ cache could be located both in memory and on the node's local disk 

+ One note here 
    + if expand this to many nodes, depends on your load balancer behavior, if it randomly distributes requests across the nodes, the same request will go to different nodes, thus increasing cache misses. 
        + could use either global caches or distributed caches for it 

## 3.2 Content Distribution Network 

+ For sites serving large amounts of static media 
+ A typical workflow 
    + A request first ask the CDN for a piece of static media 
    + CDN will serve the content if it has it locally available 
    + If not, CDN will query the back end servers for the file 
    + Then cache it locally, and serve it to the requesting user 
## 3.3 Cache Invalidation 

+ Cache needs maintenance for keeping cache coherent with the source of truth
    + if data is modified in db, should be invalidated in the cache 

+ Write Through Cache 
    + Data is written into the cache and the corresponding database at the same time 
    + It could minimize the risk of data loss, but since every write operation mush be done twice before returning success to the client, latency would be higher 

+ Write Around Cache 
    + Data is written directly to permanent storage, bypassing the cache 
    + Could reduce the cache being flooded with write operations that will not subsequently be re-read
    + But a read request for recently written data will create a cache miss 

+ Write Back Cache 
    + Data is written to cache alone
    + Write to permanent storage is done after specified intervals or under certain conditions 
    + Low latency and high throughput for write intensive applications 
    + However this speed up could cause issue of data loss in case of a crash or other adverse event because the only copy of the written data is in the cache 
    
## 3.4 Cache Eviction Policies 

+ First In First Out 
+ Last In First Out 
+ Least Recently Used 
+ Most Recently Used 
+ Least Frequently Used 
+ Randowm Replacement 

# 4. Data Partitioning 

It aims to break up a big database into many smaller parts. It's a process of splitting up a DB/ table across multiple machines to improve the manageability, performance, availability, and load balancing of an application. 

The justification for data partitioning is after a certain scale point, it's cheaper and more feasible to scale horizontally by adding more machines that to grow it vertically by adding beefier servers 

## 4.1 Partitioning Methods 

### 4.1.1 Horizontal Partitioning/ Sharding 

+ Put different rows into different tables 
+ range based partitioning as we store different ranges of data in separate tables 
+ Probelm here
    + is the range value isn't chosen carefully, the partitioning scheme will lead to unbalanced servers 
### 4.1.2 Vertical Partitioning 

+ store data related to a specific feature in their own server 
    + like photo in one server, video in another, people they follow in another 

+ not quite scalable, if our app experience some high traffic, then the single server will not be enough to handle such traffic 

### 4.1.3 Directory Based Partitioning 

+ Create a lookup service which knows your current partitioning scheme and abstracts it away from the DB access code 
+ To find a particular data entity, query the directory server that holds the mapping between each tuple key to its DB server

## 4.2 Partitioning Criteria 

+ Key or hash based partitioning 
    + apply a hash function to some key attributes of the entity we are storing 
    + need to ensure a uniform allocation of data among servers 
    + it will change the hash function when every time you add / remove some servers, the workaround is to use consistent hashing 

+ List Partitioning 
    + each partition is assigned a list of values 

+ Round robind partitioning 
+ Composite Partitioning 
    + combination of criteria above

## 4.3 Common Problems of Data Partitioning 

+ Joins and Denormalization 
    + Performing joins on a database that runs on several different servers 
    + will not be performance efficient 
    + Workaround is to denormalize database so queries perviously requiring joins can be performed from a single table 
        + but need to deal with data inconsistency issue 

+ Referential Integrity 
    + nforce data integrity constraints such as foreign keys in a partitioned database can be extremely difficult

+ Rebalancing 
    + need to do that due to 
        + data distribution is not uniform 
        + could be a lot of load on one partition 

    + In such cases, either we have to create more DB partitions or have to rebalance existing partitions, which means the partitioning scheme changed and all existing data moved to new locations. Doing this without incurring downtime is extremely difficult. Using a scheme like directory based partitioning does make rebalancing a more palatable experience at the cost of increasing the complexity of the system and creating a new single point of failure 

# 5. Indexes 

Leverage on indexes when current database performance is no longer satisfactory. Indexing could help make search faster, it could be created using one or more columns of a ddb table, providing the basis for both rapid random lookups and efficient access of ordered records. 

Index can dramatically speed up data retrieval but may itself be large due to the additional keys, which will slow down data insertion and update. 

When adding rows or making updates to existing rows for a table with an active index, we not only have to write the data but also have to update the index. This will decrease the write performance. 

This performance degradation applies to all insert, update, and delete operations for the table. For this reason, adding unnecessary indexes on tables should be avoided and indexes that are no longer used should be removed.

If the goal of ddb is often written to and rarely read from, in that case, decreasing the performance of the more common operation, which is writing, is probably not worth the increase in performance we get from reading.

# 6. Proxies

## 6.1 What is Proxy Server? 

+ Intermediate server between the client and the backend server 
+ Clients connect to proxy servers to make a request for a service like
    + web page
    + file connection 

+ Proxy server is a piece of software or hardware that acts as an intermediary for requests from clients seeking resources from other servers 
+ Proxy are used to 
    + filter requests 
    + transform requests 
        + add/ remove headers 
        + encrypt and decrypt
        + compress a resource 

    + caching 
        + if multiple clients access a particular resource, the proxy server can cache it and serve it to all clients without going to the remote server 

## 6.2 Types 

+ Open Proxy 
    + A proxy server that is accessible by any internet user 
    + type
        + anonymous proxy 
            + reveals its identity as a server but does not disclose the initial IP address 
        + transparent proxy 
            + Identify itself and with the suppot of HTTP headers 
            + IP address could be viewed 
            + main benefit of using this sort of server is its ability to cache the websites 

+ reverse proxy 
    + retrieve resources on behalf of a client from one or more servers 
    + these resources are then returned to the client, appearing as if they originated from the proxy server itself 