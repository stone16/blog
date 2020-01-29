---
title: Developing on AWS Note.5 DynamoDB
date: 2020-01-28 23:05:29
categories: Cloud
tags:
    - AWS
    - NoSQL
    - DynamoDB
top:
---

# 1. AWS Database Options 

## 1.1 SQL vs NoSQL database

 Attr | SQL | NoSQL
---|---|---
Data Storage | rows and columns | key-value, document, wide-column, graph
Schemas | fixed | dynamic
Querying | Using SQL | Focused on collection of documents
Scalability | Vertical | Horizontal 
Transactions | Supported | Support varies
Consistency | Strong | Eventual and strong 


+ Relational Database supports vertical scaling which means that a single server must be made more powerful 
+ Relational Database support ACID transactions
    + atomicity
    + consistency
    + isolation 
    + durability
+ Relational databases automatically support strong data consistency due to ACID properties of transactions.

## 1.2 AWS database Options 


Type | SQL | NoSQL
---|---|---
Transactional Databses | Amazon RDS | Amazon DynamoDB
Data Analytics/ Relationshiups | Amazon Redshift | Amazon Neptune
In-memory Data Store and Cache | | Amazon ElastiCache

+ Amazon Relational Database Service(RDS): provides relational database services in the cloud with support for the following db engines: 
    + AMazon Aurora
    + PostgreSQL
    + MySQL
    + MariaDB
    + Oracle
    + Microsoft SQL server 
+ Amazon Redshift: fast, fully managed data warehouse
    + includes Redshift Spectrum, allowing you to directly run SQL queries against exabytes of unstructured data in Amazon S3. 
+ Amazon DynamoDB: NoSQL db that supports both document and key-value store models 
+ Amazon Neptune: fully managed graph databse service
    + fully manged graoh databse service 
    + purpose-built, high-performance **graph database engine** optimized for storing billions of **relationships** and querying the graph with milliseconds latency.
    + A graph database is ideal when you need to create relationships between data and quickly query these relationships. 
    + This type of requirement is challenging to satisfy using a relational database because you would need multiple tables with multiple foreign keys. 
    + In addition, SQL queries to navigate this data would require nested queries and complex joins that could quickly become complex and inefficient as your data size grows over time. 
    + Neptune uses graph structures such as nodes (data entities), edges (relationships), and properties to represent and store data. 
    + The relationships are stored as first order citizens of the data model. 
    + This allows data in nodes to be directly linked, dramatically improving the performance of queries that navigate relationships in the data.
+ Amazon ElastiCache: **in-memory data cache** that supports a fully managed Redis or Memcached engine
    + easier to deploy, operate, and scale an in-memory data store or cache in the cloud. 
    + improve the performance of web applications by allowing you to retrieve information from fast, managed, in memory caches
    + provides 
        + redis
        + memcached

# 2. DynamoDB 

## 2.1 Intro

Amazon DynamoDB is a fast and flexible non-relational database service for all applications that need consistent, **single-digit millisecond latency** at any scale. It is a fully managed cloud database and supports both document and key-value store models.

## 2.2 Components 

+ Table 
    + data is stored in tables 
    + contain
        + item with attributes 
+ Partition
    + ddb can divide a table's items into multiple partitions based on the primary key value. 
    + an allocation of storage for a table
    + backed by SSDs and automatically replicated across multiple Availability Zones within an AWS region
    + partition key (hashkey), ddb use this to do partition 
+ sort key (range key) A sort key can be defined to store all of the items with the same partition key value **physically close together and order them by sort key value in the partition**. It represents a one-to-many relationship based on the partition key and enables querying on the sort key attribute.
+ Primary key - uniquely identify an Item
    + types
        + partition primary key
        + partition and sort primary key
+ item (400 KB at most)
    + collection of attributes 
    + not cosntrained by a predefined schema
    + items in a table can have different types of attributes 
+ attribute
    + name
    + data type
        + scalar
            + number, string, binary, boolean, null
        + multi-valued types 
            + string set
            + number set
            + binary set
        + Document types
            + List
            + Map
    + value
+ Read/ Write Consistency
    + Read
        + eventually consistent
        + strongly consistent: return most up-to-date data
        + transactional: provides ACID consistency
    + write
        + standard
        + transactional 
+ Read/ write Throughput
    + RCU: number of strongly consistent reads per second of items up to **4KB** in size 
    + WCU: number of **1KB** writes per second
+ Secondary Indexes
    + allow you to query data based on non-primary key attributes
    + contain
        + alternate key attributes
        + primary key attributes
        + optinal subset of other attributes from the base table 
    + type
        + GSI
            + queries on this index can span all the data in a table, across all partitions
            + can have different partition key and sort key from original table
            + key values do not to be unique
            + can be deleted 
            + supports eventually consistent only 
            + its own provisioned WCU and RCU
            + **queries only return attributes that are projected into the index **
        + LSI
            + index is located on the same table partition
            + sort key can be any scalar attribute 
            + cannot be deleted 
            + support eventually consitent and strong consistent 
            + use table's read and write capacity units
+ Streams
    + Ordered flow of information about changes to a table 
    + contains changes to items in a single table 
    + When you make an update to a table, DynamoDB first **persists the data durably** to the table. 
    + It then asynchronously updates the corresponding stream with information about the changes made. 
    + The asynchronous update is made to the stream with **sub-second latency**. 
    + The update to the stream does not affect the write throughput of the table
    + **stricly in the order** 
    + each change contains exactly one stream record, available for 24 hours
    + streams scale by splitting data across shards 
    + shards in detail
        + A shard is created per partition in your DynamoDB table. If a partition split is required due to too many items in the same partition, the shard gets split into children as well.
        + DynamoDB Streams captures a time-ordered sequence of item-level modifications in your DynamoDB table. This time-ordered sequence is preserved at a per shard level. In other words, the order within a shard is established based on the order in which items were created, updated or deleted. 
    + configuration
        + StreamEnabled: specify whether a stream is enabled or disabled 
        + StreamViewType: specify the information that will be written to the stream whenever data in the table is modified
            + KEYS_ONLY: only the key attributes 
            + NEW_IMAGE: entire item, as it appears after modified
            + OLD_IMAGE: entire item, as it appears before modified
            + NEW_AND_OLD_IMAGES: both the new and old images of the item
    + when a stream is created, DDB assigns an ARN(Amazon Resource Name) that can be used to retrieve information about a stream. 
+ Global table
    + A collection of one or more DynamoDB tables, all ownd by a single AWS account, identified as replica tables 
    + A replica table (or replica, for short) is a single DynamoDB table that functions as a part of a global table. Each replica stores the same set of data items.
    + data replication
        + Any changes made to any item in any replica table will be replicated to all of the other replicas within the same global table. 
        + propagate within seconds 
    + concurrent updates 
        + all replicas agree on the latest update, and converge toward a state in which they all have identical data
    + Read Consistency 
        + An application can read and write data to any replica table. 
        + If your application only uses eventually consistent reads, and only issues reads against one AWS region, then it will work without any modification. 
        + However, if your application requires strongly consistent reads, then it must perform all of its strongly consistent reads and writes in the same region. DynamoDB does not support strongly consistent reads across AWS regions; 
        + therefore, if you write to one region and read from another region, the read response might include stale data that doesn't reflect the results of recently-completed writes in the other region. 
+ Backup and Restore
    +  on-demand backup and restore capabilities 
    +  all backups in DDB work without consuming any provisioned throughput on the table 
    +  point-in-time recovery can restore the table to any point in time during last 35 days

## 2.3 APIs and operations 

### 2.3.1 Control operations 

Create and manage DynamoDB tables. Let you work with indexes, streams, and other objects that are dependent on tables. 

### 2.3.2 Data operations

Perform CRUD actions on data in a table. Also let you read data from a secondary index. 

+ PutItem
    + create a new item or replace an existing item 
+ GetItem
    + reads an item from a table
+ UpdateItem
    + edit an existing item's attributes, or adds a new item to the table
    + can perform a conditional update on an existing item 
    + can only bring some attributes instead of all comparing with putItem
+ deleteItem
    + can delete an item in a table using its primary key  

### 2.3.3 Stream operations 

Enable or disable a stream on a table, and allow access to the data modification records contained in a stream. 
### 2.3.4 Object persistence Model 

+ Allow you to persist client-side objects in DynamoDB
    + supports the mapping of objects to tables
+ Provides higher-level programming interfaces to:
    + connect to DynamoDB
    + perform CRUD operations
    + execute queries

### 2.3.5 Batch Operations 

+ BatchGetItem
    + Read up to 16MB of data consisting of up to 100 items from multiple tables
+ BatchWriteItem
    + write up to 16MB of data consisting of up to 25 put or delete requests in multiple tables 
+ retry 
    + if one request in a batch fails, the entire operation does not fail
    + retry with failed keys and data returned 

### 2.3.6 Transactional Operations

+ TransactWriteItems
    + contains a write set 
    + includes one or more PutItem, updateItem, and DeleteItem operations across
+ TransactGetItems
    + contains a read set 
    + includes one or more getItem operations across multiple tables

## 2.4 On-demand mode

Amazon DynamoDB on-demand is a flexible billing option capable of serving thousands of requests per second without capacity planning. DynamoDB on-demand offers pay-per-request pricing for read and write requests so that you pay only for what you use. 

## 2.5 Query and Scan 

+ Query 
    + reads from a table or secondary index only the items that match the primary key specified in the key condition expression.  
    + parameters
        + tableName
        + KeyContditionExpression
            + must specify partition key name and value
        + ProjectExpression 
        + ConsistentRead
        + FilterExpression
            + a string that contains conditions that DDB applies after the query operation, but before the data is returned to you 
            + all other records are discarded 
+ Scan 
    + reads all items from the table or index 
    + parameters 
        + tableName
        + ProjectionExpression 
            + a string that identify one or more attributes to retrieve from the table 
        + consistentRead
        + filterExpression 

## 2.6 Best Practices

+ Uniform workloads
+ One-To-Many tables
    + If your table has items that store a large number of values in an attribute of set type, such as string set or number set, consider removing the set attribute from the table and splitting it as separate items in another table.
    + If you frequently access large items in a table but do not use the large attribute values, consider storing frequently accessed smaller attributes in a separate table
+ Optimistic Locking with Version Number 
    + Use optimistic locking with a version number to make sure that an item has not changed since the last time you read it. 
    + Maintain a version number to check that the item has not been updated between the last read and update 

## 2.7 DynamoDB Accelerator (DAX)

+ deliver fast response times for accessing eventually consistent data 
+ DAX is a DynamoDB compatible caching service that enables you to benefit from fast in memory performance for demanding applications. It addresses three core scenarios: 
    + reduce the response times of eventually consistent read workloads by an order of magnitude, from single-digit milliseconds to microsends
    + reduce operational and application complexity by providing a managed service that is API-compatible with Amazon DynamoDB
    + DAX provides increased throughput and potential operational cost savings by reducing the need to over-provision read capacity units