---
title: AWS Lambda
date: 2020-01-29 20:18:52
categories: Cloud
tags:
    - AWS
    - Lambda
top:
---

# 1. Overview 
A compute service that lets you run code without provisioning or managing servers. It can executes your code thousands per second. You pay for the compute time you consume.

Lambda executes your code on a high-availability compute infrastructure and performs all of the administration of the compute resources, including server and operating system maintenance, capacity provisioning and automatic scaling, code monitoring and logging.

You can run AWS lambda to run your code in response to events, such as changes to data in an Amazon S3 bucket or an Amazon DynamoDB table; to run your code in response to HTTP requests using Amazon API gateway; or invoke your code using API calls made using AWS SDKs. 

# 2. Getting started

## 2.1 Configuration on console page

+ environment variables: for Lambda functions enable you to dynamically pass settings to your function code and libraries, without making changes to your code.
+ Tags: are key-value pairs that you attach to AWS resources to better organize them. 
+ Execution role: which allows you to administer security on your function, using defined roles and policies or creating new ones.
+ Basic settings: allows you to dictate the memory allocation and timeout limit for your Lambda function.
+ Network: allow you to select a VPC your function will access
+ Debugging and error handling: allow you to select a AWS [Lambda Function Dead Letter Queues](https://docs.aws.amazon.com/lambda/latest/dg/dlq.html) resource to analyze failed function invocation retries. 
+ Concurrency: allows you to allocate a specific limit of concurrent executions allowed for this function
+ Auditing and compliance: logs function invocations for operational and risk auditing, governance and compliance. 

## 2.2 Lambda concept 

Lambda can automatically scales up the number of instances of your function to handle high number of events. 

+ Function: A script or program that runs in AWS lambda. Lambda passes invocation events to your function. The function processes an event and returns a response
+ Runtimes: Lambda runtimes allow functions in different languages to run in the same base execution environment, you configure your function to use a runtime that matches your programming language. The runtime sits in between the Lambda service and your function code, relaying invocation events, context information and responses between the two. 
+ Layers: Lambda layers are a distribution mechanism for libraries, custom runtimes and other function dependencies. Layers let you manage your in-development function code independently from the unchanging code and resources that it uses. You can configure your function to use layers that you create, layers provided by AWS, or layers from other AWS customers.
+ Event source: An AWS service that triggers your function and executes its logic. 
+ Downstream resources: An AWS service that your lambda function calls once it is triggered
+ Log streams: While Lambda automatically monitors your function invocations and reports metrics to CloudWatch, you can annotate your function code with custom logging statements that allow you to analyze the execution flow and performance of your Lambda function to ensure it's working properly.

## 2.3 Programming Model 

Regardless of the language you choose, there is a common pattern ti write code for a Lambda Function includes concepts as follow:

### 2.3.1 Handler
The function AWS Lambda calls to start execution of your lambda function. When a Lambda function is invoked, Lambda starts executing your code by calling the handler function

### 2.3.2 Context 

Lambda also passes a context object to the handler function, as the second parameter. Via this context object your code can interact with Lambda

### 2.3.3 Logging 

Function can contain logging statements, it writes these logs to cloudWatch logs. 

### 2.3.4 Exceptions 
Function needs to communicate the result of the function execution to AWS Lambda. Many ways to end a request or to notify AWS Lambda an error occured during execution

### 2.3.5 Concurrency 

When your function is invoked more quickly than a single instance of your function can process events, Lambda scales by running additional instances. Each instance of your function handles only one request at a time, so you don't need to worry about synchronizing threads or processes. You can, however, use asynchronous language features to process batches of events in parallel, and save data to the /tmp directory for use in future invocations on the same instance. 

# 3. Building Lambda Functions with Java 

## 3.1 Handler Input/ Output Types 

When AWS lambda executes the Lambda function, it invokes the handler. First parameter is the input to the handler which can be event data (published by an event source) or custom input you provide such as a string or any custom data object. 

AWS Lambda supports the following input/ output types for a handler: 

+ Simple Java types (AWS Lambda supports the String, Integer, Boolean, Map and List types )
+ POJO
+ stream type 
### 3.1.1 Handler input/ output: String Type

    package example;
    
    import com.amazonaws.services.lambda.runtime.Context; 
    
    public class Hello {
        public String myHandler(String name, Context context) {
            return String.format("Hello %s.", name);
        }
    }

When you invoke a Lambda function asynchronously, any return value by your Lambda function will be ignored. Therefore you might want to ***set the return type to void*** to make this clear in your code 

### 3.1.2 Handler Input/ Output: POJO

    package example;
    
    import com.amazonaws.services.lambda.runtime.Context; 
    
    public class HelloPojo {
    
        // Define two classes/POJOs for use with Lambda function.
        public static class RequestClass {
          ...
        }
    
        public static class ResponseClass {
          ...
        }
    
        public static ResponseClass myHandler(RequestClass request, Context context) {
            String greetingString = String.format("Hello %s, %s.", request.getFirstName(), request.getLastName());
            return new ResponseClass(greetingString);
        }
    }
    
Suppose your application events generate data that includes first name and last name as shown:

    { "firstName": "John", "lastName": "Doe" }  
    
For this example, the handler receives this JSON and returns the string "Hello John Doe".

    public static ResponseClass handleRequest(RequestClass request, Context context){
            String greetingString = String.format("Hello %s, %s.", request.firstName, request.lastName);
            return new ResponseClass(greetingString);
    }

To create a Lambda function with this handler, you must provide implementation of the input and output types as shown in the following Java example. The HelloPojo class defines the handler method.

    package example;
    
    import com.amazonaws.services.lambda.runtime.Context; 
    import com.amazonaws.services.lambda.runtime.RequestHandler;
    
    public class HelloPojo implements RequestHandler<RequestClass, ResponseClass>{   
    
        public ResponseClass handleRequest(RequestClass request, Context context){
            String greetingString = String.format("Hello %s, %s.", request.firstName, request.lastName);
            return new ResponseClass(greetingString);
        }
    }
    
In order to implement the input type, add the following code to a separate file and name it RequestClass.java. Place it next to the HelloPojo.java class in your directory structure:

    package example;
            
         public class RequestClass {
            String firstName;
            String lastName;
    
            public String getFirstName() {
                return firstName;
            }
    
            public void setFirstName(String firstName) {
                this.firstName = firstName;
            }
    
            public String getLastName() {
                return lastName;
            }
    
            public void setLastName(String lastName) {
                this.lastName = lastName;
            }
    
            public RequestClass(String firstName, String lastName) {
                this.firstName = firstName;
                this.lastName = lastName;
            }
    
            public RequestClass() {
            }
        }
        

In order to implement the output type, add the following code to a separate file and name it ResponseClass.java. Place it next to the HelloPojo.java class in your directory structure:

    package example;
        
     public class ResponseClass {
        String greetings;

        public String getGreetings() {
            return greetings;
        }

        public void setGreetings(String greetings) {
            this.greetings = greetings;
        }

        public ResponseClass(String greetings) {
            this.greetings = greetings;
        }

        public ResponseClass() {
        }

    }
    
## 3.2 Context Object in Java 

When Lambda runs your function, it passes a context object to the handler. The object provides methods and properties that provide information about the invocation, function and execution environment. 

+ getRemainingTimeInMillis()
+ getFunctionName()
+ getFunctionVersion()
+ getInvokedFunctionArn() - Returns the Amazon Resource Name(ARN) used to invoke the function. Indicates if the invoker specified a version number or alias. 
+ getMemoryLimitInMB()
+ getAwsRequestId()
+ getLogGroupName()
+ getIdentity()
+ getClientContext()
+ getLogger() 
