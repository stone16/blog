---
title: 'Distributed Locks with the DynamoDB Lock Client '
date: 2021-01-14 19:51:14
categories: Cloud
tags:
top:
---


# Distributed Locks with the DynamoDB Lock Client

# 1. Overview

- DynamoDB Lock Client
    - enable you to solve distributed computing problems like leader election and distributed locking with client only code and a DDB table

- Why we need it
    - Distributed Locking is complicated
        - you need to **atomically ensure** only one actor is modifying a **stateful resource** at any given time

# 2. Practical Example

- Background
    - A retail bank that want to ensure at most one customer service representative change customer details at a time
    - solution
        - temporarily lock customer records during an update
        - suppose there are bunch different tables to contain all customer information, as the tables are independent, so we cannot just wrap the changes we need in a relational transaction
        - we need to lock customer id at a high level
        - You’d do so with a locking API action for a certain duration in your application before making any changes.

## 2.1 Locking Protocol

- For a new lock, the lock clients store a lock item in the lock table
    - it stores
        - the host name of the owner
        - the lease duration in milliseconds
        - a UUID unique to the host
        - the host system clock time when the lock was initially created

![Whole Workflow](https://i.loli.net/2021/01/15/gk6qico4zUw1YXK.png)

1. Host A acquires a lock on Moe by writing an item to the lock table on the condition that no item keyed at “Moe” exists yet. Host A
acquires the lock with a revision version number (RVN) of UUID.
2. Host B tries to get a lock on Moe with a RVN UUID.
3. Host B checks to see if a lock already exists with a GetItem call.
4. In this case, host B finds that host A holds a lock on Moe with a record version number (RVN) of UUID. The same application runs on hosts A and B. That being so, host B
expects host A to heartbeat and renew the lock on Moe in less than 10 seconds, if host A intends to keep the lock on Moe. Host A heartbeats once, and uses a conditional update on the lock keyed at Moe to update the RVN of the lock to UUID.
5. Host B checks 10 seconds after the first AcquireLock call to see if the RVN in A’s lock on Moe changed with a conditional UpdateItem call and a RVN of UUID.
6. Host A successfully updates the lock. Thus, host B finds the new RVN equal to UUID and waited 10 more seconds. Host A died after the first heartbeat, so it never changes the RVN past UUID. When host B calls tries to acquire a lock on Moe for the third time, it finds that the RVN was still UUID, the same RVN retrieved on the second lock attempt.
7. In this case, hosts A and B run the same application. Because host B expects host A to heartbeat if host A is healthy and intends to keep the lock, host B considers the lock on Moe expired. Host B’s conditional update to acquire the lock on Moe succeeds, and your application makes progress!

# Reference

1. [https://aws.amazon.com/blogs/database/building-distributed-locks-with-the-dynamodb-lock-client/](https://aws.amazon.com/blogs/database/building-distributed-locks-with-the-dynamodb-lock-client/)