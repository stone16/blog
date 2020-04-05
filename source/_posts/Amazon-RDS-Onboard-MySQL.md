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

+ Create DB instance as prerequisite 
+ Use MySQL client application or utility to connect to the instance
+ Specify DNS address from the DB instance endpoint as the host parameter, specify the port number from DB instance endpoint as the port parameter 
+ For endpoint, we could find in AWS console, on the "connectivity & security" tab 
+ To  connect from MySQL client, using command shown as below
    + `mysql -h mysql–instance1.123456789012.us-east-1.rds.amazonaws.com -P 3306 -u mymasteruser -p`
+ Amazon RDS creates an SSL certificate for your DB instance when the instance is created     + you could do it with native password or with IAM authentication 
    + `mysql -h mysql–instance1.123456789012.us-east-1.rds.amazonaws.com --ssl-ca=rds-ca-2015-root.pem -p` 
+ we could also connect from MySQL workbench 
    + See instructions on [Connnecting from MySQL Workbench](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html) 

## 2.3 Configure high availability for a production DB instance 

Amazon help you do this with Multi-AZ deployments. SQL server DB instances use SQL server Database Mirroring or Always On Availability Groups. 

In a Multi-AZ deployment, Amazon RDS automatically provisions and maintains a synchronous standby replica in a different Availability Zone. The primary DB instance is synchronously replicated across Availability Zones to a standby replica to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups. Running a DB instance with high availability can enhance availability during planned system maintenance, and help protect your databases against DB instance failure and Availability Zone disruption.

The high availability feature is not scaling solution for read-only scenarios, you cannot use a standby replica to serve read traffic. 

NOTICE -- DB instances using Multi-AZ deployments can have increased write and commit latency compared to a Single-AZ depolyment, due to the synchrounous data replication that occurs. 

In the event of a planned or unplanned outage of your DB instance, Amazon RDS automatically switches to a standby replica in another Availability Zone if you have enabled Multi-AZ. The time it takes for the failover to complete depends on the database activity and other conditions at the time the primary DB instance became unavailable. Failover times are typically 60–120 seconds. However, large transactions or a lengthy recovery process can increase failover time. When the failover is complete, it can take additional time for the RDS console to reflect the new Availability Zone.

## 2.4 Configure a DB instance in VPC 

+ VPC is a virtual network that is logically isolated from other virtual networks in the AWS cloud. Amazon VPC lets you launch AWS resources into a VPC. 
+ VPC must have at least 2 subnets. And those subnets must be in two different AZs 
+ If you want DB instance in the VPC to be publicly accessible, you must enable the VPC attributes DNS hostnames and DNS resolution 

## 2.5 Configure MySQL database parameters and features 

+ Manage DB engine configuration by associating DB instances with parameter groups. A DB parameter group act as a container for engine configuration values that are applied to one or more DB instances 
+ For MySQL, AWS has memcached support 

## 2.6 Modify a DB instance running the MySQL database engine 

+ We could change the settings of a DB instance to add additional storage or changing the DB instance class 
+ Notice: some cahnges will result in an outage because Amazon RDS must reboot DB instance for the change to take effect 
    + We could either modify through AWS console
    + Or through AWS CLI
    + Or through RDS API 
+ we could do settings as follow:
    + Allocate storage 
    + Auto minor version upgrade 
    + Backup retention period 
        + number of days that automatic backups are retained 
    + Backup window 
    + Certificate Authority 
    + Database port 
    + DB engine version 
    + DB instance class 
    + DB parameter group 
    + Delete protection 
        + prevent your DB from being deleted  
    + Enhanced Monitoring 
    + IAM DB authentication 
    + Kerberos authentication 
    + License Model 
    + Log Exports 
        + We could publish Database logs to Amazon Cloudwatch logs 
    + Maintenance window 
    + Multi-AZ deployment 
    + Performance Insight 
    + Processor features 
    + Provisioned IOPS 
    + Storage auto scaling 
    + Subnet group 

## 2.7 Configure database backup and restore 

+ Amazon RDS creates and saves automated backups of your DB instance
+ RDS creates a storage volume snapshot of DB instance, backing up entire DB instance 
+ The first snapshot of a DB instance contains the data for the full DB instance. Subsequent snapshots of the same DB instance are incremental, which means that only the data that has changed after your most recent snapshot is saved.
+ [Guide for backup and restore an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_CommonTasks.BackupRestore.html)

## 2.8 Monitor a MySQL DB instance 

### 2.8.1 Overview 

+ We should store historical monitoring data, the stored data will gove a baseline to compare against with current performance data
+ With Amazon RDS, you could monitor network throughput, I/O for read and write, metadata operations, client connections
+ Some adviced metrics 
    + High CPU or RAM consumption 
    + Disk space consumption 
    + Network traffic 
    + Database connections 
    + IOPS metrics 

+ Monitoring Tools 
    + Amazon RDS Events 
        + subscribe to events thus could be notified when changes occur with a DB instance 
    + Database log files
    + Amazon RDS Enhanced Monitoring
        + Look at the metrics in real time for the operating system
    + Amazon CloudWatch Metrics 
    + Amazon CloudWatch Alarms
    + Amazon CloudWatch Logs 
    + In RDS console, you could see: 
        + the number of connections to a DB instance 
        + the amount of read and write operations to a DB instance 
        + the amount of storage that a DB instance is currently utilizing 
        + the amount of memory and CPU being utilized for a DB instance 
        + the amount of network traffic to and from a DB instance 
+ Amazon RDS Metrics 
    + CPUCreditUsage 
        + The number of CPU credits spent by the instance for CPU utilization. One CPU credit equals one vCPU running at 100 percent utilization for one minute or an equivalent combination of vCPUs, utilization, and time.
    + DatabaseConnections
    + DiskQueueDepth 
        + the number of outstanding IOs waiting to access the disk
    + FailedSQLServerAgentJobsCount 
        + the numebr of failed SQL server agent jobs during the last minute 
    + ReadIOPS 
        + the average number of disk read I/O operations per second
    + ReadLatency 
        + the average amount to time taken per disk I/O operation  

### 2.8.2 Enhanced Monitoring 

Real time metrics for the operating system

CloudWatch gathers metrics about CPU utilization from the hypervisor for a DB instance, and Enhanced Monitoring gathers its metrics from an agent on the instance. As a result, you might find differences between the measurements, because the hypervisor layer performs a small amount of work. The differences can be greater if your DB instances use smaller instance classes, because then there are likely more virtual machines (VMs) that are managed by the hypervisor layer on a single physical instance. Enhanced Monitoring metrics are useful when you want to see how different processes or threads on a DB instance use the CPU


# Reference 
1. https://docs.aws.amazon.com/AmazonRDS 
2. https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html

