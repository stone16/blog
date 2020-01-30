---
title: AWS essential(5) - Architecting
date: 2020-01-29 21:06:50
categories: Cloud
tags:
    - AWS
top:
---
# 1. Introduction to well-architected framework 

+ developed through reviewing customers' architectures on AWS 
+ build and deploy faster 
+ lower or mitigate risks 
+ make informed decision 
+ learn aws best practices 


## 1.1 Five Pillars 

### 1.1.1 Operational Excellence 
+ Run and monitor systems to deliver business value 
+ continually improve supporting processes and procedures 

Disign Principles 

+ perform operations as code 
+ annotate documentation 
+ make frequent, small, reversible changes 
+ refine operations procedures frequently 
+ anticipate failure 
+ learn from all operational failures 


### 1.1.2 Security 

+ protect information, systems, assets 
+ do risk assessments and mitigation strategies 

Design Principles 

+ Implement a strong identity foundation 
+ enable traceability 
+ apply security best practices 
+ automate security best practices 
+ protect data in transit and at rest 
+ prepare for security events


### 1.1.3 Reliability 
+ recover from infrastructure or service disruptions 
+ dynamically acquire computing resources to meet demand 
+ mitigate disruptions 

Design Principles:

+ test recovery procedures 
+ automatically recover from failure
+ scale horizontally to increase aggregate system availability 
+ stop guessing capacity 
+ manage change in automation 


### 1.1.4 Performance Efficiency

+ use computing resources efficiently 
+ maintain efficiency as demand changes and technologies evolve 

Design Principles 

+ Democratize advanced technologies 
+ Go global in minutes 
+ use serverless architectures 
+ Experiment more often 
+ mechanical sympathy 

### 1.1.5  Cost Optimization 

+ avoid or eliminate unneeded cost or suboptimal resources 
Design Principles 

+ adopt a consumption model 
+ measure overall efficiency 
+ stop spending money on data center operations 
+ analyze and attribute expenditure 
+ use managed services to reduce cost of ownership 


# 2. Fault tolerance and highly available architecture 

## 2.1 Fault tolerance 

+ remain operational even if components fail 
+ build-in redundancy of an application's components

## 2.2 High Availability 

+ always functioning and accessible 
+ downtime is minimized 
+ without human intervention 

## 2.3 High availability tools 

### 2.3.1 elastic load balancers 

elb - distribute income traffic to your hosts, also send metrics to cloudwatch 

### 2.3.2 elastic IP addresses 

static IP addresses designed for dynamic cloud computing, able to mask failures 

applications still accessible 

### 2.3.3 amazon route 53

Authorized DNS server, 

### 2.3.4 auto scaling 

launches and terminates by specified conditions 

can be adjusted/ modified 

create new resources on demand 

### 2.3.5 amazon cloudwatch 

collects and tracks infromations 

## 2.4 Fault tolerant tools 

1. Amazon simple queue service 
2. Amazon simple storage service 
3. Amazon simple DB 
4. Amazon relational database service 

# 3. Web hosting 
## 3.1 benefits 
+ cost effective 
handle peak capacity 

do on demand provisioning 

+ scalable 
+ on demand 

## 3.2 Web hosting services 

Products to assist transition: 

1. Amazon Virtual Private Cloud 
2. Amazon Route 53 
3. Amazon CloudFront 
4. Elastic load balancing 
5. Firewalls/ AWS shield
6. Auto scaling 
7. App servers/ EC2 instances 
8. Amazon ElasticCache 
9. Amazon RDS/ Amazon DynamoDB 

## 3.3 Key Architectural considerations 

1. No more physical network appliances 
2. Firewalls everywhere 
3. consider multiple data centers 