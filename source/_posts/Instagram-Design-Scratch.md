---
title: Instagram Design Scratch
date: 2020-11-15 09:59:24
categories: SystemDesign
tags:
top:
---
# 1. Intro and Instagram 

+ social networking service 
+ upload and share photos and videos 
+ share info either publicly or privately 
+ share through other social networking platforms 
    + facebook
    + twitter
    + flickr
    + tumblr


+ requirement 
    + functional 
        + users should be able to upload/ download/ view photos 
        + search based on photo/ video titles 
        + users can follow other users 
        + system will generate and display a user's News Feed consisting of top photos from all the people the user follows 

    + non-functional 
        + services need to be highly available 
        + latency come to be 200ms for News Feed Generation 

# 2. Design

+ patterns
    + read heavy system 
        + need to effectively manage storage 
        + 100% reliable of data 
        + low latency 


+ system design 
    + we need to support two scenarios
        + upload photos 
        + view/ search photos

    + db 
        + image storage 
        + image metadata storage 


+ Separate read and write servers 
    + web servers have a connection limit 
    + assume max is 500 connections, that means server cannot have more than 500 concurrent uploads or reads 

+ reliability 
    + run a redundant secondary copy of the service that is not serving any traffic if only one instance of a service is required to run at any point 

+ data sharding 
    + partition based on UserID?
        + hot user 
        + non-uniform storage as some users may upload a lot of images 
        + what if one shard is full? 
        + unavailability -- all users data come to be unavailable all of sudden 

    + use photoId as partition key 

+ news feeds - e.g 100 feeds 
    + real time generation 
        + need to go to user follow table to get all of its following people 
        + then query with user id to get their 100 updates 
        + combine them, go through our ranking algo 
        + then generate the output 

    + should choose pre-generating way as it will save a lot of times 
        + have dedicated servers that are continuously generating users' News Feeds and storing them in a UserNewsFeed table

+ approaches for seanding news feed contents to users 
    + pull
        + clients can pull the News Feed contents from the server on a regular basis or manually whenever they need it 
            + problems
                + new data might not be shown to the users until clients issue a pull request 
                + most of time, pull requests response could be empty 

            + push 
                + servers can push new data to the users as soon as it is available 
                + users then need to maintain a long poll request with the server for receiving the updates 
                + issue 
                    + user has follow a lot of people 
                    + celebrity user who has meillions of followers 
                    + server then have to push updates quite frequently 

            + hybrid 
                + move all the users who have a high number of follows to a pull based model and only push data to those users who have a few hundred follows 