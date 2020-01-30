---
title: Elasticsearch Overview
date: 2020-01-29 20:35:39
categories: Cloud
tags:
    - Elastic Search
top:
---
# 1. Overview 

## 1.1 Intro

+ Full-text search
+ Analytics engine 
+ Can analyze big volumes of data in near real time 
+ used as the underlying engine/ tech that powers applications that have complex search features and requirements. 

## 1.2 Use cases 

1. online web store - allow customers to search for products. - use Elasticsearch to store entire product catelog and inventory. Provide search and autocomplete suggestions for them. 
2. collect log or transaction data - mine data to look for trends, statistics, summarizations, or anomalies 
3. price alert platform - scrape vendor prices, push them into Elasticsearch and use its reverse-search (Percolator) capability to match price movements against customer queries and eventually push the alerts out to the customer once matches are found
4. analytics/business-intelligence needs and want to quickly investigate, analyze, visualize, and ask ad-hoc questions on a lot of data (think millions or billions of records). In this case, you can use Elasticsearch to store data and then use Kibana to build custom dashboards that can visualize aspects of data. Additionally, you can use the Elasticsearch aggregations functionality to perform complex business intelligence queries against your data.

## 1.3 Basic concepts 

### 1.3.1 NRT 

Near Realtime: it only has a slight latency from the time you index a document until the time it becomes searchable. 
### 1.3.2 Cluster 

A cluster is a collection of one or more nodes **(servers)** that together **holds your entire data** and provides **federated indexing and search capabilities** across all nodes.

### 1.3.3 Node 

A node is a single server that is part of your cluster, stores your data, and participates in the cluster's indexing and search capabilities. 

### 1.3.4 Index 

An index is a collection of documents that have somewhat similar characteristics. For example, you can have an index for customer data, another index for a product catalog, and yet another index for order data. 

An index is **identified by a name** (that must be all lowercase) and this name is used to refer to the index when performing indexing, search, update, and delete operations against the documents in it.

### 1.3.5 Document 

A document is a basic unit of information that can be indexed. For example, you can have a document for a single customer, another document for a single product, and yet another for a single order. This document is expressed in JSON (JavaScript Object Notation) which is a ubiquitous internet data interchange format.

Within an index/type, you can store as many documents as you want. Note that although a document physically resides in an index, a document actually must be indexed/assigned to a type inside an index.

### 1.3.6 Shards & Replicas 

An index can potentially store a large amount of data that can exceed the hardware limits of a single node. For example, a single index of a billion documents taking up 1TB of disk space may not fit on the disk of a single node or may be too slow to serve search requests from a single node alone.

To solve this problem, Elasticsearch provides the ability to **subdivide your index into multiple pieces called shards.** When you create an index, you can simply define the number of shards that you want. Each shard is in itself a fully-functional and independent "index" that can be hosted on any node in the cluster.

Sharding brings you several benefits: 

1. Allow you to horizontally **split/ scale content volume** 
2. Allow you to **distribute and parallelize** operations across shards thus increasing performance/ throughput 

Mechanics of how a shard is distributed and how its documents are aggregated back into search reqeusts are completely managed by elasticSearch. 

To summarize, each index can be split into multiple shards. An index can also be replicated zero (meaning no replicas) or more times. Once replicated, each index will have primary shards (the original shards that were replicated from) and replica shards (the copies of the primary shards).

# 2. Benefits 

1. search faster compared with database 
2. During an indexing operation, Elasticsearch converts raw data such as log files or message files into internal documents and stores them in a basic data structure similar to a JSON object. Each document is a simple set of correlating keys and values: the keys are strings, and the values are one of numerous data types—strings, numbers, dates, or lists.

# 3. Where do the Files come from? 

Elasticsearch uses Lucene under the hood to handle the indexing and querying on the shard level, the files in the data directory are written by both Elasticsearch and Lucene. 

Lucene is responsible for writing and maintaining the L**ucene index files while Elasticsearch writes metadata related to features on top of Lucene**, such as field mappings, index settings and other cluster metadata – end user and supporting features that do not exist in the low-level Lucene but are provided by Elasticsearch.

# Reference 
1. https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html 
2. http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html 
3. https://www.elastic.co/blog/found-dive-into-elasticsearch-storage
