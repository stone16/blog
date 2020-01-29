---
title: Developing on AWS Note.7 API Gateway
date: 2020-01-28 23:14:08
categories: Cloud
tags:
    - Gateway
    - AWS
    - RESTFul
top:
---
# 1. What is Amazon API Gateway? 

## 1.1 Functionalities
+ enables developers to create, publish, maintain, monitor and secure APIs
+ allow you to connect your applications to AWS services and other public or private websites
+ provides consistent RESTFUL APIs for mobile and web applications to access AWS services and other resources hosted outside of AWS
+ Handles all the tasks involved in **accepting and processing** up to hundreds of thousands of concurrent API calls, including **traffic management, authorization and access control, monitoring and API version management**

## 1.2 Use cases

+ Create a unified API frontend for multiple microservices
+ DDoS protection and throttling for backend 
+ Authenticate and authorize requests to a backend
+ Throttle, meter, and monetize API usage by third party developers 

+ message transformation and validation
    + **models** can be created to define a schema for reqeust/ response messages
    + A **Mapping Template** can then be used to transform data from one model to another
    + request/ response payload and header can be validated against the model
    + message transformation and mapping can be done using API Gateway
    + customers will often map request messages to a canonical format for downstream applications using API Gateway.  --> **transform a response body from the backend data format to the frontend data format**
+ Expose backend resources
    + allow you to create an API that acts as a front door for applications to access data, business logic or functionality from your backend service
    + expose
        + HTTP endpoints
        + AWS services
        + AWS Lambda functions
+ Increase API performance : Cache
    + eploy APIs to Regional or Edge-optimized endpoints to bring them closer to their clients. Cache API responses to the API Gateway response cache.
    + You can also enable API caching in Amazon API Gateway to cache your endpoint’s response. 
    + With caching, you can reduce the number of calls made to your endpoint and also improve the latency of the requests to your API. When you enable caching for a stage, API Gateway caches responses from your endpoint for a specified time-to-live (TTL) period, in seconds. 
    + API Gateway then responds to the request by looking up the endpoint response from the cache instead of making a request to your endpoint
+ Control Access to APIs
    + method level throttling 
    + client usage throttling and quota limits sepcified in a usage plan 
    + help prevent one customer from consuming all of your backend system’s capacity
+ Secure API method invocations
    + creating a resource policy 
        + a JSON policy document that you attach to an API to control whether a specified principal(IAM user or role) can invoke the API 
        + You can use a resource policy to enable users from a different AWS account to securely access your API or to allow the API to be invoked only from specified source IP address ranges or Classless Inter-Domain Routing (CIDR) blocks.
    + Creating IAM permission policy, can protect: 
        +  the creation, deployment, and management of an API
        +  the invocation of the methods in the API and refresh of its cache
    + Creating a Private API endpoint that can only be accessed by a VPC client
    + Integrating with Amazon Cognito or Lambda authorizers to authenticate and authorize clients before accessing backend resources
    + Resource policy and IAM permission capabilities offer flexible and robust access controls that can be applied to an entire API set or individual methods. 
# 2. Best practices 

## 2.1 Developing an API

+ WHen API client requests come from the same region where the API is deployed, choose a regional API endpoint type
+ Test invking the API before deploying it
+ Use HTTP 500 error code for error handling 
+ Cache only GET methods 

# 3. Serverless Application Model (SAM)

Template driven development model for defining serverless apps

+ supports 
    + Lambda
    + API Gateway
    + DynamoDB table
    + Any resource that AWS CloudFormation supports 