---
title: AWS essential(4)-Security
date: 2020-01-29 21:05:42
categories: Cloud
tags:
    - AWS
    - Security
top:
---

# 1. Introduction to AWS security 

+ Approach to security 
    + resilient infrastructure 
    + high security 
    + strong safeguards 
+ Controls 
+ AWS products and features 


+ Network Security 
    + build in firewalls 
    + encryption in transit 
    + private dedicated connections 
    + ddos mitigation 

+ Data Encryption 
    + encrption capabilities for aws storage/ database 
    + key management options 
    + hardware based cryptographic key storage options 

+ Access Control and management 
    + Identity and access management 
    + Multifactor authentication 
    + integration and federation with corporate directories 

+ Monitoring and Logging 
    + deep visibility into API calls 
    + log aggregation and options 
    + alerts 

# 2. The AWS Shared Responsibility Model 

Share responsibility for securing data. 

AWS responsible of --- security of the cloud 

+ compute 
+ storage 
+ databse 
+ networking 


Customer responsible of ---- security in the cloud 

+ what to store 
+ which aws services 
+ location 
+ content format 

# 3. AWS Access Control and Management 

## 3.1 IAM overview - 
### 3.1.1 Functions
+ Control access to AWS resources 
+ authentication
    + who can access resources
    + use AWS IAM policy 
+ Authorization 
    + how they can use resources 

Manage accesses to: 

+ compute 
+ storage 
+ database 
+ application services 

### 3.1.2 Roles 

+ User
+ Group 
+ Permissions 
+ Role 

### 3.1.3 Features 

+ Shared access to your AWS account 
+ Granular permissions 

You can grant different permissions to different people for different resources. 

+ secure access to AWS resources for applications that run on Amazon EC2 

You can use IAM features to securely provide credentials for applications that run on EC2 instances. These credentials provide permissions for your application to access other AWS resources. 

+ Multi-factor authentication(MFA)
+ Identity federation 

You can allow users who already have passwords elsewhere—for example, in your corporate network or with an internet identity provider—to get temporary access to your AWS account.

### 3.1.4 functionalities 

+ manage users and their access 
+ manage roles and their permissions 
+ manage federated users and their permissions 


## 3.2 How IAM works 

### 3.2.1 Elements contained

+ Resources 
    + The user, role, group and policy objects that are stored in IAM. 
+ Identities 
    + The IAM resource objects taht are used to identofy and group.  

[Understanding how IAM works](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html)
