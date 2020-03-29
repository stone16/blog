---
title: 'Amazon RDS Onboard - MySQL '
date: 2020-03-28 20:39:50
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

## 2.1 Manage security for DB instance 

### 2.1.1 Security Overview 

+ Run DB instance in a vertual private cloud based on the Amazon VPC service 
+ Use AWS Identity and Access Management policies to assign permissions that determine who is allowed to manage Amazon RDS resrouces 
+ Use security groups to control what IP addresses or Amazon EC2 instances can connect to your databases on a DB instance
+ Use SSL or TLS connections with DB instances [Instructions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html)
+ Use Amazon RDS encryption to secure DB instances and snapshots at rest. It used AES-256 encryption algorithm to encrypt data on the server that hosts DB instance

### 2.1.2 Manage access with Policies - resource level 
+ A poloicy is an object that associated with an identity or resource, defines their permissions. 
+ An IAM administrator could use policies to specify who has access to AWS resources, and what actions they can perform on the resources 

### 2.1.3 Access control in DB instance level - security group 

+ Security groups control the access that traffic has in and out of a DB instance 
    + VPC security groups 
    + DB security groups
    + EC2-classic security groups 
+ VPC security group
    + enable a specific source to access a DB instance in a VPC in the security group 
    + source could be: 
        + a range of addresses 
        + another VPC security group 
+ DB security group 
    + Used with DB instances that are not in a VPC and on the EC2 classic platform  
    + DB security group rules apply to inbound traffic only 
    + You don't need to specify port number or protocol when adding rules 

## 2.2 Connect to DB instance 

## 2.3 Configure high availability for a production DB instance 

## 2.4 Configure a DB instance in VPC 

## 2.5 Configure MySQL database parameters and features 

## 2.6 Modify a DB instance running the MySQL database engine 

## 2.7 Configure database backup and restore 

## 2.8 Import and export data 

## 2.9 Monitor a MySQL DB instance 

## 2.10 Replicate data 

# Reference 
1. https://docs.aws.amazon.com/AmazonRDS 
2. https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html



