---
title: Designing a URL Shortening service like TinyURL
date: 2020-11-08 16:03:55
categories: SystemDesign
tags:
top:
---
# 1. Thoughts 

Based on System Design Guide, our thoughts could follow the framework described [here](https://llchen60.com/System-Design-General-Guides-from-Grokking-SDI/)

Based on that: 

+ Requirement Clarification 
    + what feature we need
        + create new tinyUrl 
        + read existing tinyUrl, parse it to websites url 
+ Envelope estimation 
    + what traffic amount it would be? Read/ Write Ratio? 
    + Compute storage size we need
    + Network bandwidth usage 

+ System Interface Design 
    + generateNewTinyUrl
        + original url 
        + output parsed url 
    + parseTinyUrl()
        + input tiny url 
        + output - mapped original url 

+ Data Model 
    + how data will flow between different components of the system 

+ High level design and detailed design 

+ bottle neck resolving 

# 2. Detail 

## 2.1 Requirements 

+ Functional 
    + shorten given url 
    + access a short url, user will be redirected to original link 
    + users should optionally be able to pick a custom short link for their url 
    + links will expire after a standard timespan, users should be able to specify the expiration time 

+ Non-Functional Requirements 
    + System should be highly available 
    + Url redirection should happen in real time with minimal latency 
    + Shortened links should not be guessable 

+ Extended Requirements
    + analytics - how many times a redirection happen
    + should also be accessible through REST APIs by other services 

## 2.2 Capacity Estimation and Constraints

+ read heavy system 
    + estimate read write ratio 100:1 

+ traffic estimates 
    + suppose 500M new URL shortening request 
    + then based on ratio, could be 50B read request 
    + QPS
        + for write 
            + 500MM/ (30days * 24 hours * 3600 seconds) = ~ 200 URL/s
        + for read 
            + 200 * 100 = 20K/s

+ storage estimate 
    + how long do we want to store them? 
        + what's the total number of objects we want to store? 
            + suppose 5 years 
            + 500MM * 5 years * 12 months = 30 billion 
            + suppose every object is about 500 bytes, then in total 
                + 30B * 500 bytes = 15 TB

+ bandwidth estimate 
    + for write 
        + 200 * 500 bytes/ s = ~100KB/s

    + for read 
        + 20K * 500 bytes = ~ 10MB/s 

+ memory estimates 
    + we may want to cache some hot URLs that are frequently accessed, 20/ 80 rule 
    + cache 20% hot URLs for a day 
    + memory usage 
        + 0.2 * 1.7 B * 500 bytes = ~170GB 

## 2.3 System APIs 

+ createURL (clientId, originalUrl, userName, customAlias, expireDate)
    + clientId could be used for throttling based on allocated quota 
    + return 
        + shortened url 

+ deleteURL (clientId, urlKey)

## 2.4 DB design 

+ need two table, one store info for the URL mappings; one for the user's data about who created the short link 

## 2.5 System Design and Algorithm 

+ How to generate a short and unique key for a given URL 
    + Encoding actual URL 
        + compute a unique hash of the given URL 
        + hash could be encoded for display 
        + If we use MD5 algorithm as our hash function, it'll produce a 128 bit hash value
        + After base64 encoding, we'll get a string having more than 21 characters
            + take the first 8 letters for the key 
                + for duplication, choose some other characters out of the encoding string or swap some characters 

    + generate keys offline 
        + have a standalon key generation service that generates random six letter strings beforehand and stores them in a database
        + whenever we want to shorten a URL, we will take one of the already generated keys and use it 

## 2.6 Deta Partitioning and Replication 

+ Range based partitioning
    + Store URLs in separate partitions based on the hash key's first letter
    + could lead to unbalanced DB servers  
+ Hash Based Partitioning 
    + use hash, and calculate which partition to use based on that 

## 2.7 Cache 

+ could use memcache or other similar in memory cache solution 
+ how muach cache memory should we have?
    + 20% of daily traffic 

+ what cache eviction policy should we use?
    + LRU 

## 2.8 Load Balancer 

+ We could choose to add LB at: 
    + between clients and application servers
    + between application servers and database servers
    + between apllication servers and cache servers 

## 2.9 Purging / DB cleanup 

+ We should not actively search for expired links to remove them, as it would put a lot of pressure on our db 
    + slowlyremove and do a lazy cleanup

## 2.10 Metrics and Security 

+ Record metrics 
    + country of visitor
    + data and time 
    + web page that referred the click 
    + browser 
    + platform
    + etc. 

+ security and permissions 
    + store the permission level with each URL in the database 