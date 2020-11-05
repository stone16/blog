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

# 7. Redundancy and Replication 

+ redundancy 
    + duplication of critical components or functions of a system with the intention of increasing the reliability of the system 
        + backup
        + fail saft 
        + direct improvement on actual system performance 


+ replication 
    + shareing information to ensure consistency between redundant resources
        + to improve reliability
        + fault tolerance
        + accessibility 

    + primary replica relationship 
        + primary server gets all the updates 
        + then ripple through to the replica servers 
        + each replica outputs a message stating that it has received the update successfully 
# 8. SQL vs NoSQL

## 8.1 Concepts
+ Relational databases 
    + structured 
    + predefiend schemas 

+ Non-relational database 
    + unstrutured 
    + distributed
    + dynamic schema 


+ SQL
    + store data in rows and columns 
    + each row contains:
        + all the info about one entity 

    + each column contains:
        + all separate data points 

+ NoSQL
    + Key-Value Stores 
        + store in an arry of key value pairs
        + key is an attribute name which is linked to a vlue 
            + redis
            + voldemort
            + dynamo

    + document database
        + data is stored in documents (instead of rows and columns in a table)
        + documents are grouped together in collections 
        + each document can have an entirely different structure

    + wide column databases
        + have column families, which are containers for rows 
        + no need to know all the columns up front and each row doesn't have to have the same number of columns
        + best suited for analyzing large datasets 
        + type 
            + HBase
            + Cassandra

    + Graph Database
        + used to store data whose relations are best represented in a graph 
        + data is saved in graph structures with:
            + nodes 
                + entities
            + properties
                + information about the entities 
            + lines 
                + connections between the entities


## 8.2 Differences between SQL and NoSQL

+ Storage
    + SQL 
        + each row represents an entity 
        + each column represents a data point about the entity 

    + NoSQL
        + could be key value
        + document
        + graph 

+ Schema
    + SQL
        + each record conforms to a fixed schema 
            + columns must be decided and chosen before data entry 
            + each row must have data for each column 
        + schema modification need to involve modifying the whole database and go offline 

    + NoSQL
        + schemas are dynamic 
        + columns can be added on the fly, and each row doesn't have to contain data for each column

+ Query 
    + SQL 
        + Structured query language for defining and manipulating the data 

    + NoSQL 
        + Query focus on a collection of documents 
        + UnQL - unstructured query language 


+ Scalability
    + SQL
        + vetically scalable 
            + by increase the horsepower (memory, CPU, etc.) of the hardware 

    + NoSQL
        + horizontally scalable 
            + we could add more servers easily in database infrastructure to handle more traffic 
            + any cheap hardware could host NoSQL database

+ Reliability or ACID comliancy
    + ACID
        + atomocity
        + consistency
        + isolation
        + durability 

    + most NoSQL solutions sacrifice ACID compliance for performance and scalability 


## 8.3 Choose which one? 

+ Reasons for use SQL
    + Need ACID compliance 
        + ACID reduces anomalies and protects the integrity of your db by prescribing exactly how transactions interact with the database 
    + Data is structured and unchanging 
 

+ Reasons for use NoSQL 
    + Store large volumens of data that often have little to no structure 
        + NoSQL allows use to add new types 
        + with document based databases, you can store data in one place without having to define what types of data those are in advance 

    + Making the most of cloud computing and storage 
        + cloud based storage requires data to be easily spread across multiple servers to scale up 

    + Rapid development 



# 9. CAP Theorem 

CAP states it's impossible for a distributed software system to simultaneously provide more than two out of three of the following gurantees:
+ consistency 
+ availability 
+ partition tolerance 

CAP say when designing a distributed system we could only pick two of them: 

+ consistency 
    + all nodes see the same data at the same time 
    + consistency is achieved by updating several nodes before allowing further reads 

+ availability 
    + every requests get a response on success/ failure 
    + availability is achieved by replicating the data across different servers 

+ partition tolerance 
    + system continues to work despite msg loss or partial failure 
    + data is sufficiently replicated across combinations of nodes and networks to keep the system up through intermittent outages 



We cannot build a general data store that is continually available, sequentially consistent, and tolerant to any partition failures. We can only build a system that has any two of these three properties. Because, to be consistent, all nodes should see the same set of updates in the same order. But if the network loses a partition, updates in one partition might not make it to the other partitions before a client reads from the out-of-date partition after having read from the up-to-date one. The only thing that can be done to cope with this possibility is to stop serving requests from the out-of-date partition, but then the service is no longer 100% available.


# 10. Consistent Hashing 

## 10.1 Existing Hash Funciton 
Distributed Hash Table is super important in distributed scalable systems. Hash Tables need a key, a value, and a hash function where hash function maps the key to a location where the value is stored. 

A common thought of hash function would be key%n, but it has several drawbacks:

1. Not horizontally scalable 
    1. whenever a new cache host is added to the system, all existing mappings are broken 

2. Not load balanced 
    1. expecially for non-uniformly distributed data 
    2. some caches would come to be hot and saturated while the others idel and are almost empty 

## 10.2 Consistent Hashing 

+ want to minimize reorganization when nodes are added or removed 
+ when the hash table is resized  only k/n keys need to be remapped where k is the total number of keys and n is the total number of servers 
+ objects are mapped to the same host if possible 


+ how it works 
    + map a key to an integer
    + all integers are placed on a ring such that the values are wrapped around 
    + each object is assigned to the next server that appears o nthe circle in clockwise order  --> provide an even distribution of objects to servers 
    + if a server fails and is removed from the circle, only the objects that were mapped to the failed server need to be reassigned to the next server in clockwise order 

# 11. Long-Polling, WebSockets, Server-Sent Events 

Long Polling, WebSockets and Server Sent events are popular communication protocols between a client like web browser and a web server 

## 11.1 Ajax Polling 

+ Client repeatedly polls a server for data
+ Client make a request and wait for the server to respond with data. If no data available, an empty response is returned 

+ whole workflow
    + client opens a connection and requests data from the server using regular HTTP 
    + the requested webpage sends requests to the server at regular intervals 
    + the server calculates the response and sends it back, like regular HTTP traffic 
    + the client repeats the above three steps periodically to get updates from the server 


+ pitfall
    + Polling let client continue to ask server for any new data, as a result, a lot of responses are empty, creating HTTP overhead 

## 11.2 HTTP Long-Polling 

+ Workflow 
    + If the server does not have any data available for the client, instead of sending an empty response, the server holds the request and waits until some data becomes available 
    + once available, a full response is sent to the client. Client then immediately request information from the server so that the server will almost always have an available waiting request that it can use to deliver data in response to an event 


+ The client makes an initial request using regular HTTP and then waits for a response.
+ The server delays its response until an update is available or a timeout has occurred.
+ When an update is available, the server sends a full response to the client.
+ The client typically sends a new long-poll request, either immediately upon receiving a response or after a pause to allow an acceptable latency period.
+ Each Long-Poll request has a timeout. The client has to reconnect periodically after the connection is closed due to timeouts.


## 11.3 WebSockets

+ Provides Full duplex communication channels over a single TCP connection. 
+ Provides a persistent connection between a client and a server that both parties can use to start sending data at any time. 
+ lower overhead, real time data transfer 
    + it provides a standardized way for the server to send content to the browser without being asked by the client and allowing for messages to be passed back and forth while keeping the connection open 
+ Workflow 
    + Client establish a websocket connection through a process known as the WebSocket handshake
    + if the process succeeds server and client can exchange data in both directions at any time 

## 11.4 Server Sent Events - SSEs

+ Client establish a persistent and long term connection with the server 
+ Server use this connection to send data to a client 
+ But client would need another tech/ protocol to send data to the server 

# Reference 

1. https://en.wikipedia.org/wiki/Consistent_hashing#:~:text=In%20computer%20science%2C%20consistent%20hashing,is%20the%20number%20of%20slots. 