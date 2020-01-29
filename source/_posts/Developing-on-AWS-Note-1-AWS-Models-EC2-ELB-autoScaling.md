---
title: 'Developing on AWS Note.1  - AWS Models, EC2, ELB, autoScaling'
date: 2020-01-28 22:58:39
categories: Cloud
tags:
    - AWS
    - EC2 
    - ELB
    - AutoScaling
top:
---
# 0. Overview 

We use SDKs to interract with Application Programing Interface(API), and then connect to all AWS services. 



# 1.  Cloud computing definition 
+ enable you to stop thinking of your infrastructure as hardware, and instead think of it and use it as software. 
    
# 2. Models of Cloud Computing 

+ IaaS (Infrastructure as a Service)
    + basic buiding blocks for Cloud IT 
        + Networking features 
        + Computers 
        + Data storage space 
    + PaaS (Platform as a Service)
        + enables you to run applications without the need to manage underlying infrastructure(hardware and operating systems)
    + SaaS (Software as a Service)
        + A complete product that is run and managed by the service provider
# 3. AWS Service Stack 

+ Infrastructure 
    + Regions 
    + Availability Zones 
    + Edge Locations 
+ Foundation Services 
    + Compute
        + virtual instances 
        + auto scaling 
        + load balancing 
    + networking 
    + storage 
        + object 
        + block 
        + archive 
+ Platform Services 
    + Compute
        + AWS Lambda
        + AWS Elastic Beanstalk 
        + Amazon ECS
        + Amazon EKS 
    + database
        + relational 
        + No SQL 
        + Caching 
        + Products 
            + DynamoDB
            + RDS - relational database service 
            + Elastic Cache 
            + Redshift - data warehouse, for analysis and migration 
    + Analytics 
        + Cluster computing 
        + real time 
        + data warehouse 
        + data workflows 
        + Products 
            + EMR - managed hadoop framework 
            + Kinesis 
            + CloudSearch 
            + ElasticSearch 
    + App services 
        + Queuing 
        + Orchestration 
        + App streaming 
        + Transcoding 
        + Email 
        + Search 
        + Products 
            + SQS 
            + SNS 
            + SES 
            + Amazon Step Functions 
    + Deployment and management 
        + containers 
        + Dev/ ops tools 
        + resource templates 
        + usage tracking 
        + monitoring and logs 
        + products 
            + CodeCommit 
            + CodeDeploy 
            + CodePipeline 
            + CodeBuild 
            + X-Ray 
    + Mobile Services 
        + identity 
        + sync 
        + mobile analytics 
        + notifications 
        + products 
            + Cognito 
            + Pinpoint 
            + API gateway 
+ Applications 
    + Virtual Desktops 
    + Collaboration and Sharing 

# 4. Compute services 

## 4.1 EC2 
+ Computers in the cloud. 
+ Can create images of your servers at any time with a few clicks or simple API call. 
+ different instance type for different use cases:
    + low traffic websites 
    + small database 
    + high performance web services 
    + high performance databases 
    + distributed memory caches 
    + data warehousing 
    + log or data-processing applications 
    + 3D visualizations 
    + Machine learning 
+ Pricing 
    + on demand 
    + reserved instances 
    + spot instances 

## 4.2 ELB - Elastic Load Balancing  

+ distribute traffic across multiple EC2 instances, in multiple Availability Zones 
+ Support health checks to detect unhealthy Amazon EC2 instances 
    + To discover the availability of instances, a ELB periodically sends pings, attempts connections or sends requests to test the EC2 instances.  
+ Supports the routing and load balancing of traffic to Amazon EC2 instances. 
+ when the LB determins that an instance is unhealthy, it stops routing requests to that instance. 
+ sticky sessions
    + enables the load balancer to bind a user's session to a **specific server instance**. 
+ we should get rid of sticky sessions since: 
    +  limit application's scalability 
    +  lead to unequal load across servers 
    +  affect end-user response time since a single user's load isn't even spread across servers. 

+ Instead of using sticky sessions: cache 
    + manage user sessions by
        + store locally to the node responding to the HTTP request 
        + designate a layer which can store those sessions in a scalable and robust manner. 
    + Duration based session stickiness
        + LB uses a special LB generated cookie to rack the application instance for each request. **When the load balancer reveives a request, it first checks to see whether this cookie is present in the request**.  If so, the reqeust is sent to the application instance specified in the cookie. If not, the LB chooses an application instance based on the existing load balancing algo. **A cookie is inserted into the response for binding subsequent requests from the same user to that application instance**. The stickiness policy configuration defines a cookie expiration, which establishes the duration of validity for each cookie. Cookie will be automatically updated after its duration expires. 
    + Application base session stickiness 
        + LB uses a **special cookie** to associate the session with the original server that handled the reqeust. But follows the lifetime of the application-generated cookie corresponding to the cookie name specified in the policy configuration. 
        + The LB only inserts a new stickiness cookie if the application response includes a new application cookie. 
        + The load balancer stickiness cookie does not update with each request. If the application cookie is explicitly removed or expires, the session stops being sticky until a new application cookie is issued.
        + Application often store session data in memory, but this approach does not scale well
+ Methods available to manage session data without sticky sessions include: 
    + using ElasticCache to store session data 
    + using Amazon DynamoDB to store session data 

## 4.3 Auto Scaling  

Auto Scaling helps you ensure that you have the correct number of EC2 instances available to handle the load for your application. Auto Scaling is particularly well-suited for applications that experience hourly, daily, or weekly variability in usage.

# 5. Exceptions and Errors handle 

+ 400 series: handle error in application 
+ 500 series: retry operations 

Java SDK throes the following unchecked(runtime) exceptions when error occur:

+ AmazonServiceException
    + indicates that the reqeust was correctly transmitted to the service, but for some reason, the service was not able to process it, and returned an error response instead. 
+ AmazonClientException 
    + indicates that a problem occured inside the hava client code 
        + try to send a request to AWS 
        + try to parse a response from AWS 
+ IllegalArgumentException 
    + throw if you pass an illegal argument when performing an operation on a service  