---
title: System Design General Guides - from Grokking SDI
date: 2020-10-08 17:55:45
categories: SystemDesign
tags:
top:
---
# 1. Requirements Clarifications 

+ Ask about the exact scope of the problem 
    + For a twitter like system design 
        + will users of our service be able to post tweets and follow others? 
        + Create and display user's timeline?
        + Tweets contain photos and videos?
        + Front end design too?
        + Searchable tweets?
        + Display hot trending topics?
        + Push notification for new tweets? 


# 2. Back of the envelope estimation 

+ estimate the scale of the system we are going to design 
    + could help later when we will be focusing on scaling, partitioning, load balancing and caching
        + what scale is expected 
            + number of new tweets
            + number of tweet views
            + number of timeline generations per sec 

        + how much storage will we need
            + videos and photos will play important role on our estimation 

        + what network bandwidth usage are we expecting
            + crucial in deciding how we will manage traffic and balance load between servers 


# 3. System Interface Definition 

Establish exact contract expected from the system 
```
postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, …)  

generateTimeline(user_id, current_time, user_location, …)  

markTweetFavorite(user_id, tweet_id, timestamp, …)  
```

# 4. Define Data Model 

Define the data model, to clarify how data will flow between different components of the system. Later, will guide for data partitioning and management. 

User: UserID, Name, Email, DoB, CreationData, LastLogin, etc.
Tweet: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.
UserFollow: UserID1, UserID2
FavoriteTweets: UserID, TweetID, TimeStamp

# 5. High level design 

+ Draw a block diagram with 5 - 6 boxes representing the core components of our system. We need to identify enough components that are needed to solve the actual problem from end2end. 
+ For twitter 
    + Multiple application servers to serve all the read/ write requests with load balancers in front of them for traffic distributions 
    + we could have separate servers for handling specific traffic (like much more read than write)
    + Efficient database that can store all the tweets and can support a huge number of reads 
    + Also need a distributed file storage system for storing photos and videos 


# 6. Detailed Design 

+ Dig deeper to 2 or 3 components 
    + present different approaches
    + pros and cons 
    + explain why we prefer one on the other 
    + consider tradeoffs between different options while keeping system contraints in mind 


+ Question Examples
    + Since we will be storing a massive amount of data, how should we partition our data to distribute it to multiple databases? Should we try to store all the data of a user on the same database? What issue could it cause?
    + How will we handle hot users who tweet a lot or follow lots of people?
    + Since users’ timeline will contain the most recent (and relevant) tweets, should we try to store our data in such a way that is optimized for scanning the latest tweets?
    + How much and at which layer should we introduce cache to speed things up?
    + What components need better load balancing?

# 7 Identifying and resolving bottlenecks 

+ Discuss as many bottlenecks as possible and different approaches to mitigate them 
    + Any single point of failure in our system? 
    + Do we have enough replicas of the data so that if we lose a few servers, we can still serve our users? 
    + Do we have enough copies of different services running such that a fewe failures will not cause a total system shutdown?
    + How to monitor performance of our service? 