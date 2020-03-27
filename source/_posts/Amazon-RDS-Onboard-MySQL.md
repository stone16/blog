---
title: 'Amazon RDS Onboard - MySQL '
date: 2020-03-26 20:39:50
categories: Cloud
tags:
    - RDS
    - MySQL
    - AWS
top:
---

# 1. Overview 

Amazon Relational Databazse Service 
+ provide cost efficient, resizable capacity for relational database and manage common database administration tasks 
+ CPU, memory, storage, IOPS can be scaled independently 
+ help you manage backups, software patching, automatic failure detection, and recovery 
+ Automated backups 
+ Can get high availability with a primary instance and synchronous secondary instance that you can fail over to when problems occur.
+ Integrate with IAM and VPC settings

## 1.1 How does amazon help you do the setup? 

Overall, you control your database by using DB instance, you could select different kind of host with different configuration, AWS will help you to deploy it in your selected region, and it will be automatically deployed to different AZ, to increase availability. Let's go through it in detail. 

### 1.1.1 DB Instances 

+ An isolated database env in the AWS Cloud 
+ One instance can contain multiple user-created databases 
+ Each DB instance runs one DB engine, for DB engine, we mean MySQL, MariaDB, PostgreSQL, etc. 
+ You could change your selected computation and memory capacity by using [DB instance class](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html).
+ You could choose to use multiple availability zones, AZ is engineered to be isolated from failures in other AZs, by launching instances in separate AZs, you can protect your applications from the failure of a single location
### 1.1.2 Basic setup 

+ AWS account 
+ IAM user 

# 2. MySQL on RDS 


# Reference 
1. https://docs.aws.amazon.com/AmazonRDS 
2. https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html


