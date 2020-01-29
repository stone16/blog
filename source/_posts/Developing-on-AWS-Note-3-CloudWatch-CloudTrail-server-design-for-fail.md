---
title: 'Developing on AWS Note.3  - CloudWatch, CloudTrail, server-design for fail'
date: 2020-01-28 23:02:31
categories: Cloud
tags:
    - AWS
    - CloudWatch
    - CloudTrail
top:
---

# 1. CloudWatch 

##  Why use Amazon CloudWatch? 

Need it to: 

+ monitor CPU, memory, disk I/O, network  -> metrics
+ react to application log events and availability -> logs/ event
+ automatically scale ec2 instance fleet -> logs/ event
+ view operational status and identify issues -> alarms, dashboard 


Actually, we could use Amazon CloudWatch to gain **system-wide visibility** into **resource utilization**, **application performance**, and **operational health**. You can use these insights to react and keep your application running smoothly. Amazon CloudWatch monitors your AWS Cloud resources and your cloud-powered applications. It tracks the metrics so that you can visualize and review them. You can also set alarms that will fire when a metric goes beyond a limit that you specified. CloudWatch gives you visibility into resource utilization, application performance, and operational health.

# 2. CloudTrail 

CloudTrail is integrated with several AWS services. 
+ EC2
+ VPC
+ S3
+ EBS
+ DDB
+ RDS
+ Redshift
+ CloudFormation 
+ IAM
+ ...etc. 


AWS CloudTrail is an AWS service that generates logs of calls to the AWS API. AWS CloudTrail can **record all activity** against the services it monitors. Here are questions that you can answer using CloudTrail logs: **who, when, what, which, where**? While the coverage is extensive, not all services are covered in CloudTrail logs. You can use the AWS API **call history produced by CloudTrail to track changes to AWS resources**, including creation, modification, and deletion of AWS resources such as Amazon EC2 instances, Amazon VPC security groups, and Amazon EBS volumes.


You can use the CloudTrail console to view the last 90 days of recorded API activity and events in an AWS region. You can also download a file with that info, or a subset of info based on the filter and time range you choose

# 3. Best practices of developing cloud apps

+ consider designing applications that are **loosely coupled**. 
    + think of your application as a consumer and provider of services 
    + design and develop app as **granular components** that can be delivered and scaled independently. 
+ Architect for resilience; 
    + Set up your servers to scale automatically based on the number of users concurrently visiting your application.
    + Autoscaling would enable your application to handle a surge in volumn during a sale or propmotion and go back to normal 
    + set up a **cluster of nodes** such that when one node fails, another node automatically picks up all the traffic. 
    + Consider setting up **read replicas for your database**. 
+ design for failure
    + In case of service failure, your application may log the failure and retry at a later time.
    + If the service is slow to respond, your application could retry by using an exponential backoff algorithm: retry after increasing amounts of time between attempts.
        + This approach attempts to **reach the service without overwhelming it** with repeated requests and potentially aggravating the latency issue. 
+ log metrics and monitor performance 
+ implement a strong DevOps model
    + Operationalize the development and deployment process for your application
    + Develop a **modular, automated, and continuous build** process. 
    + Ensure **consistency** in the development, staging, and production environments. Set up scripts or use robust tools to consistently configure your environments.
+ implement security in every layer 
    + infrastructure
    + application
    + data at transit and at rest 
    + user authentication and authorization


