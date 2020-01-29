---
title: 'Developing on AWS Note.6 AWS Serverless platform, Lambda'
date: 2020-01-28 23:06:41
categories: Cloud
tags:
    - AWS
    - Lambda
    - Serverless
top:
---
# 1. Serverless Computing 
## 1.1 Benefits
+ with serverless deployment and operation, only need to
    + build and deploy apps
    + monitor and maintain apps
+ no need to provision, scale and manage any servers 

## 1.2 Use cases

+ web applications 
    + automatically scale up and down 
    + run in a highly available configuration across multiple data centers 
+ backends
    + build serverless backends using AWS lambda to handle web, mobile, internet of Things(IoT), and 3rd party APR requests 
+ mobile backends
+ data processing 
    + execute code in response to triggers 
        + changes in data 
        + shifts in system state
        + actions by users

# 2. AWS serverless Platform

+ Compute
    + AWS lambda
    + AWS Fargate
+ API Proxy
    + Amazon API Gateway
    + AWS AppSync
+ Storage
    + Amazon S3
+ Database
    + Amazon DynamoDB
    + Amazon Aurora
+ Interprocess Messaging
    + Amazon SNS
    + Amazon SQS
+ Orchetration
    + AWS Step Functions 
+ Analytics
    + Amazon Kinesis
    + Amazon Athena
+ Developer Tools
    + Frameworks
    + SDKs
    + Libraries


Serverless applications don’t require provisioning, maintaining, and administering servers for backend components such as compute, databases, storage, stream processing, message queueing, and more. You also no longer need to worry about ensuring application fault tolerance and availability.


# 3. AWS Lambda

## 3.1 What is AWS Lambda?

+ Compute service that enables you to run code without provisioning or managing servers. 
+ Pay only for the compute time you consume 
+ Run code for virtually any type of application or backend service, all with zero administration. 
+ can set up code to automatically trigger from other AWS services or call it directly from any web or mobile app

## 3.2 Concepts
+ Event source - what triggers the call
    + Used to pass in event data to the handler 
    + java/C# supports simple data types and stream input/ output
    + includes all of the data and metadata Lambda needs
+ Context object
    + provides handler runtime information 
    + interact with Lambda execution environment 
    + contain
        + AWS requestId - Used to track specific invocations of a Lambda function
        + Remaining time - The amount of time in milliseconds that remain before your function timeout occurs
        + logging - Each language runtime provides the ability to stream log statements to Amazon CloudWatch Logs.
+ Language choice
+ Execution environment - permissions and resources
+ Runtime 
    + a program that runs a lambda function's handler method when the function is invoked
    + can include a runtime in your function's deployment package in the form of an executable file named bootstrap
    + responsible for running the function's setup code
    + read the handler name
    + read invocation events from the runtime API
    + runtime passes the event data to the function handler, and posts response from the handler back to Lambda
+ handler function
    + When a Lambda function is invoked, code execution begins at what is called the handler. The handler is a specific code method (Java, C#) or function (Node.js, Python) that you’ve created and included in your package. 


## 3.3 Using Lambda

+ Bring own code 
    + bring own libraries
    + custom runtimes 
+ Simple resource model 
    + CPU and network allocated proportionately
+ Flexible Use
    + Synchronous/ Asynchronous
    + Integrated with other AWS services
+ Flexible Authorization
    + securely grant access to resources and VPCs 
    + Fine grained control for invoking your functions 

## 3.4 How it works

Function can be invoked by

+ push model
    + event based invocation 
    + event sources invoke your Lambda function 
    + e.g
        + S3, SNS, Cognito, Echo 
+ request-response invocation 
    + causes Lambda to execute the function **synchronously** and returns the response immediately to the calling application. This invocation type is available for custom applications
+ pull event model
    + Lambda polls the event source and invokes function when it detects an event
    + E.G
        + DynamoDB, SQS, Kinesis 

## 3.5 Develop and deploy workflow

+ create a lambda handler class in code
+ create lambda function 
+ allow Lambda to assume an IAM role
+ upload the code
+ invoke the AWS Lambda Function 
+ Monitor function 

## 3.6 Lambda Layers

+ Centrally manage code and data that is shared across multiple functions
    + reduce size of deployments
    + speed up deployment 
    + Limits
        + 5 layers
        + 250 MB
    
+ Layer
    + ZIP archive that contain libraries, a custom runtime, or other dependencies
    + with layers, you can use libraries in your function without needing to include them in deployment package
    + extracted to the /opt directory in the function execution env 
    + use AWS Serverless Application Model (AWS SAM) to manage layers and your function's layer configuration

## 3.7 Best practices

+ Function Code
    + Separate the Lambda handler (entry point) from your core logic 
        + can make a more unit-testable function 
    + take advantage of Execution Context reuse
        + make sure any externalized configuration or dependencies that your code retrieves are stored and referenced locally after initial execution
        + Limit the re-initialization of variables/objects on every invocation. Instead use static initialization/constructor, global/static variables and singletons. Keep alive and reuse connections (HTTP, database, etc.) that were established during a previous invocation.
    + use environment variables
    + control the dependencies in your function's deployment package
    + minimize the complexity of your dependencies 
        + Prefer simpler Java dependency injection frameworks like Dagger or Guice, over more complex ones like Spring Framework
    + avoid using recursive code 
    + share common dependencies with layers
+ Function Configuration
    + performance testing your Lambda function for memory
        + crucial part in ensuring you pick the optimum memory size configuration
        + Any increase in memory size triggers an equivalent increase in CPU available to your function. 
        + The memory usage for your function is determined per-invoke and can be viewed in AWS CloudWatch logs.
    + Load test Lambda Function
        + Determine an optimum timeout value
        + Important to analyze how long your function runs so that you can better determine any problems with a dependency service that may increase the concurrency of the function beyond what you expect
        + This is especially important when your Lambda function makes network calls to resources that may not handle Lambda's scaling.