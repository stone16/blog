---
title: AWS X-Ray Intro(Java)
date: 2020-01-29 20:34:38
categories: Cloud
tags:
    - AWS
    - Distributed Tracing
top:
---
Fancy tech! You now can trace the performance not only for request and response, also about calls that your application makes to downstream AWS resources, microservices, databases and HTTP web APIs with AWS X-ray! 


# 1. Overview 
## 1.1 What does X-ray do? 
+ service that collects data about requests that your application serves
+ provide tools to view, filter, gain insights into that data 
+ Can see detailed information not only about the request and response, but also about calls that your application makes to downstream AWS resources, microservices, databases and HTTP web APIs. 


## 1.2 X-ray SDK provides

+ Interceptors: add to your code to trace incoming HTTP requests 
+ Client Handlers: instrument AWS SDK clients that your application uses to call other AWS services 
+ An HTTP clinet to use to to instrument calls to other internal and external HTTP web services 

## 1.3 Mechanism 

SDK sends JSON segment documents to a daemon process listening for UDP traffic. ***The X-Ray daemon buffers segments in a queue and uploads them to X-Ray in batches***

X-Ray use the trace data to generate a detailed service graph, which shows the client, your front-end service, and backend services that your front-end service calls to process requests and persist data.

# 2. Use cases and requirement 

+ Use X-Ray SDK and AWS service integration to instrument requests to your applications that are running locally or on AWS compute services. 
+ X-Ray SDK records data about **incoming and outgoing requests** and sends it to the X-Ray daemon(relays the data in batches to X-Ray)

+ When your application calls DDB to retrieve user information from a DDB table, the X-Ray SDK records data from both **the client request and the downstream call** to DynamoDB. 
+ Service Integration 
    + Add tracing headers to incoming requests 
    + send trace data to X-Ray 
    + run the X-Ray daemon 

+ Java Integration 
    + Add a servlet filter 
+ Supported AWS services
    + AWS lambda 
    + Amazon API Gateway 
    + Elastic Load Balancing 
    + AWS Elastic ZBeanstalk 

+ Code and configuration changes 
    + Detailed tracing of front-end and downstream calls requires only **minimal changes to build and deploy-time configuration**.
    + **AWS resource configuration**: Change AWS resource settings to **instrument requests to a Lambda function**. Run the X-Ray daemon on the instances in your Elastic Beanstalk environment by changing an option setting
    + **Build configuration**: Take X-Ray SDK for Java submodules as a **compile-time dependency** to instrument all downstream requests to AWS services, and to resources such as Amazon DynamoDB tables, Amazon SQS queues, and Amazon S3 buckets.
    + **Application configuration**: To instrument incoming HTTP requests, add a **servlet filter** to your Java application
    + **Class or object configuration** – To instrument outgoing HTTP calls in Java, import the X-Ray SDK for Java version of HttpClientBuilder instead of the Apache.org version.
    + **Functional Changes**: Add a **request handler **to an AWS SDK client to instrument calls that it makes to AWS services. Create subsegments to group downstream calls, and add debug information to segments with annotations and metadata.

# 3. Concepts 
## 3.1 Segments 

The compute resources running your application logic send data about their work as **segments**. A segment provides the resource's name, details about the request, and details about the work done. For example, when an HTTP request reaches your application, it can record the following data about:

+ Host 
+ Request 
+ Response 
+ Work done
    + start time 
    + end time
    + subsegments 
+ issues that occur 

## 3.2 Subsegments 

A segment can break down the data about the work done into subsegments. Subsegments provide more granular timing information and details about downstream calls that your application made to fulfill the original request. A subsegment can contain additional details about a call to an AWS service, an external HTTP API, or an SQL database. You can even define arbitrary subsegments to instrument specific functions or lines of code in your application.

## 3.3 Service Graph 

X-Ray uses the data that your application sends to generate a service graph. Each AWS resource that sends data to X-Ray appears as a service in the graph. Edges connect the services that work together to serve requests. Edges connect clients to your application, and your application to the downstream services and resources that it uses.

A service graph is **a JSON document** that contains information about the services and resources that make up your application. The X-Ray console uses the service graph to generate a visualization or service map.

## 3.4 Traces 

A trace ID **tracks the path of a request through whole application**. A trace collects all the segments generated by a single request. That request is typically an HTTP GET or POST request that travels through a load balancer, hits your application code, and generates downstream calls to other AWS services or external web APIs. 

The **first supported service** that the HTTP request interacts with adds a trace ID header to the request, and propagates it downstream to track the latency, disposition, and other request data.

## 3.5 Sampling 

To ensure efficient tracing and provide a representative sample of the requests that your application serves, the X-Ray SDK applies a sampling algorithm to determine which requests get traced. By default, the X-Ray SDK records the **first request** each second, and **five percent** of any additional requests.

## 3.6 Tracing Header 

All requests are traced, up to a configurable minimum. After reaching that minimum, a percentage of requests are traced to avoid unnecessary cost. The sampling decision and trace ID are **added to HTTP requests in tracing headers named X-Amzn-Trace-Id**.

# 4. AWS X-Ray Daemon 

Daemon is a software application, listening for traffic on UDP port 2000, gather **raw segment data**, and relays it to the AWS X-ray API. 

You need give Daemon enough permissions to send data to X-Ray. 

# 5. Working with Java 

The X-Ray SDK for Java provides a class named **AWSXRay** that provides the global recorder, a **TracingHandler** that you can use to instrument your code. You can configure the global recorder to customize the **AWSXRayServletFilter** that creates segments for incoming HTTP calls.

## 5.1 Sampling Rules 

SDK uses the sampling rules deined in the X-Ray console to determine which requests to record. The default rule traces the first request each second, and five percent of any additional requests across all services sending trances to X-Ray. 

## 5.2 Tracing Incoming requests with the X-Ray SDK 

Use the X-Ray SDK to **trace incoming HTTP requests **that your application serves on an EC2 instance in Amazon EC2, AWS Elastic Beanstalk, or Amazon ECS. 

Use a **Filter** to instrument incoming HTTP requests. When you add the X-Ray servlet filter to your application, the X-Ray SDK for Java **creates a segment for each sampled request**. This segment includes timing, method, and disposition of the HTTP request. Additional instrumentation creates subsegments on this segment.

Each segment has a name that identifies your application in the service map. The segment can be named statically, or you can configure the SDK to name it dynamically based on the host header in the incoming request. Dynamic naming lets you group **traces based on the domain name in the request**

The message handler creates a segment for each incoming request with an http block that contains the following information:

+ HTTP method
+ Client address 
+ Response code 
+ Timing 
+ User agent 
+ Content length 

## 5.3 Tracing AWS SDK calls with the X-Ray SDK for Java 
When your application **makes calls to AWS services to store data**, write to a queue, or send notifications, the X-Ray SDK for Java **tracks the calls downstream in subsegments**. Traced AWS services and resources that you access within those services (for example, an Amazon S3 bucket or Amazon SQS queue), appear as downstream nodes on the service map in the X-Ray console.

The X-Ray SDK for Java **automatically instruments all AWS SDK clients when you include the aws-sdk and aws-sdk-instrumentor submodules in your build**. If you don't include the Instrumentor submodule, you can choose to instrument some clients while excluding others.

E.G: instrument an AmazonDynamoDB client, pass a tracing handler to AmazonDynamoDBClientBuilder

    import com.amazonaws.xray.AWSXRay;
    import com.amazonaws.xray.handlers.TracingHandler;
    
    ...
    public class MyModel {
      private AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
            .withRegion(Regions.fromName(System.getenv("AWS_REGION")))
            .withRequestHandlers(new TracingHandler(AWSXRay.getGlobalRecorder()))
            .build();
            
## 5.4 Tracing Calls to Downstream HTTP Web Services with the X-Ray SDK for Java 

When your application makes call to microservices or publis HTTP APIs, you can use the X-Ray SDK for java's version of **HTTPClient** to instrument those calls and add the API to the service graph as a downstream service. ** Use the xray HttpClientBuilder **

## 5.5 Custom subsegments with the X-Ray SDK for Java 

Subsegments extend a trace's segment with details about work done in order to serve a request. Each time you make a call with an instrumented client, the X-Ray SDK records the information generated in a subsegment. You can create additional subsegments to group other subsegments, to measure the performance of a section of code, or to record annotations and metadata.

To manage subsegments, use the beginSubsegment and endSubsegment methods.

    import com.amazonaws.xray.AWSXRay;
    ...
      public void saveGame(Game game) throws SessionNotFoundException {
        // wrap in subsegment
        Subsegment subsegment = AWSXRay.beginSubsegment("Save Game");
        try {
          // check session
          String sessionId = game.getSession();
          if (sessionModel.loadSession(sessionId) == null ) {
            throw new SessionNotFoundException(sessionId);
          }
          mapper.save(game);
        } catch (Exception e) {
          subsegment.addException(e);
          throw e;
        } finally {
          AWSXRay.endSubsegment();
        }
      }
      
## 5.6 Add Annotations and Metadata to Segments with the X-Ray SDK for Java 

 You can add annotations and metadata to the segments that the X-Ray SDK creates, or to custom subsegments that you create.
 
 Annotations are key-value pairs with string, number, or Boolean values. Annotations are indexed for use with filter expressions. Use annotations to record data that you **want to use to group traces** in the console, or when calling the GetTraceSummaries API.
 
 Metadata are key-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions. **Use metadata to record additional data that you want stored in the trace but don't need to use with search**.