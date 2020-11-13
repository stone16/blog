---
title: Designing Pastebin
date: 2020-11-12 20:46:26
categories: SystemDesign
tags:
top:
---
# 1. Intro and requirements

+ Pastebin
    + service enable users to store plain text or images over the network and generate unique urls to access the uploaded data 


+ requirements
    + functional 
        + users could upload/ paste their data and get a unique URL to access it 
        + users only able to upload text 
        + data and links will expire after a specific timespan, user should be able to specify expeiration time 
        + users should optionally be able to pick a custom alias for their paste 

    + non functional 
        + system should be highly reliable, no data loss any time 
        + system should be highly available
        + small latency 
        + paste links should not be guessable 


# 2. thoughts 
+ requirement clarification 
    + what's the accept types of the file? 
    + what's the maximum size of the file? 
        + 10MB

+ hardware requirement 
    + read write request number 
    + average request size 
    + storage size 
        + we probably don't want to use more than 70% of our total storage capacity at any point 
    + cache size 

+ how to achieve highly reliable, and highly available 
    + duplicate host 

+ how you want to store those file 
    + of course key value
        + then question come to be what's the overall number of key we need to store 

+ high level design 
    + besides normal client, application server, and object storage 
    + we could have another db to store metadata, like paste id, user 