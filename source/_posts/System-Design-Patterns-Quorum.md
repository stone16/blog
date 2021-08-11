---
title: System Design Patterns - Quorum
date: 2021-08-10 21:26:01
categories: SystemDesign
tags:
top:
---

# 1. Background

In distributed system, data is replicated across multiple servers for fault tolerance and high availability.

Once system decides to maintain multiple copies of data, another problem arises: how to make sure that all replicas are consistent?? 

# 2. Dive Deep

## 2.1 Definition

- Quorum
    - Minimum number of servers on which a distributed operation needs to be performed successfully before declaring the operation's overall success

## 2.2 How it works?

- Suppose a database is replicated on 5 machines, then quorum refers to the minimum number of machines that perform the same action for a given transaction in order to decide the final operation for that transaction
- So in a set of 5, three machines form the majority quorum, quorum **enforces the consistency requirement** needed for distributed operations
- Quorum Number
    - N / 2 + 1
- Quorum is achieved when nodes follow the below protocol R + W > N
    - R  minimum read nodes
    - W minimum write nodes
    - N  nodes in the quorum group

## 2.3 Where is it used?

- Chubby
    - Use paxos for leader election, which use quorum to ensure strong consistency
- Cassandra
    - Ensure data consistency, each write request can be configured to be successful only if the data has been written to at least a quorum of replica nodes
- Dynamodb
    - Writes to a sloppy quorum of other nodes in the system
    - All read/ write operations are performed on the first N healthy nodes from the preference list, which may not always be the first N nodes encountered walking the consistent hashing ring