---
title: 'Developing on AWS Note.8 SQS, SNS'
date: 2020-01-28 23:17:05
categories: Cloud
tags:
    - AWS
    - SQS
    - SNS
    - Notification Service
    - Queue
top:
---
# 1. Why use a queuing service? 

Consider a scenario where an application produces messages that must be processed by a consumer downstream. The producer needs to know how to connect to the consumer. If the consumer fails for some reason, then messages may be lost. If new consumer instances are launched to recover from failure or to keep up with an increased workload, the producer needs to be explicitly made aware of the new consumer instances. In this scenario, the producer is tightly coupled with the consumers and the coupling is prone to brittleness.

In this way, there will be a strong interdependency between teh consumer and the producer, which is a **tightly coupled system**. which is not fault tolerant, if any one component in our system fails, the entire system will fail. 

+ Having a queue service decouples the producer from the consumer.
    + queue is a temprary repositiory for messages that are awaiting processing. 
    + acts as a buffer between the component producing data and the component receiving the data for processing. 
    + A queue supports multiple producers and consumers interacting with the same queue. 
    + A single queue can be used **simultaneously** by many distributed application components, with no need for those components to coordinate with each other to share the queue. A queue delivers each message at least once.
    + In this way, a producer can put messages on the queue regardless if they are being read by the consumer or not. 

# 2. Developing with Amazon Simple Queue Service (Amazon SQS)

## 2.1 Types

+ Standard queues
    + message ordering is not guranteed
    + message may be duplicated 
    + maximum throughput 
+ FIFO queue
    + message ordering is preserved
    + message only receive once
    + limited throughput (300 transactions per second)

## 2.2 Used to solve tightly linked systems

+ problem to be solved
    + An example of image processing: the sequential operations of uploading, storing, and encoding the image, creating a thumbnail, and copyrighting are tightly linked to each other. This tight linkage complicates the recovery operations when there has been a failure.
+ queuing chain pattern
    + Achieve loose coupling of systems by using queues between systems and exchanging messages that transfer jobs
    + This enables asynchronous linking of systems.
    + lets you increase the number of virtual servers that receive and process the messages in parallel. 
    + If there is no image to process, you can configure auto scaling to terminate the servers that are in excess.

## 2.3 Operations 

### 2.3.1 Client 
+ sendMessage 
    + send message to a specific queue
    + max size: 256 KB 
    + parameters of a sendMessage operation
        + QueueUrl: specify the url of the queue that the message should be sent to
        + MessageBody: specify the message to send 
        + DelaySeconds: specify the number of seconds to delay a specific message. Messages will become available for processing after the delay time is finished. 
        + MessageAttributes 
            + specify structured metadata about the message
                + timestamp
                + signature
                + geospatial data

+ receiveMessage
    + specify short polling or long polling 
    + When requesting to get a message from the queue, **you cannot specify which message to get**. You simply specify the maximum number of messages you want to get (up to 10), and Amazon SQS returns up to that maximum number.
    + parameters 
        + WaitTimeSeconds
        + MaxNumberOfMessages
        + VisibilityTimeout
            + period of time that a message is invisible to the rest of your application after an application component gets it from the queue.
            + prevents multiple components from processing the same message 
            + during the visibility time, the component that received the message usually processes it and then delete it from the queue. 
            + this prevents multiple components from processing the same message 
    + polling types
        + short polling
            +  Amazon SQS samples a subset of the servers (based on a weighted random distribution) and returns messages from only the sampled servers
            +  if you keep retrieving from your queues, SQS samples all the servers, and you will eventually receive all of your messages. 
            +  occurs when the WaitTimeSeconds parameter of a ReceiveMessage call is set to 0 or the queue attribute ReceiveMessageWaitTimeSeconds is 0
        + long polling
            + better and preferred way to retrieve messages 
            + if your application has a single thread polling multiple queues, switching from short polling to long polling will likely not work, because the single thread will wait for the long poll timeout on any empty queues, delaying the processing of any queues which may contain messages. 
            + Amazon SQS long polling doesn’t return a response until a message arrives in the queue or the long poll times out 
            + inexpensive 
            + unless the connection time out, the response to the ReceiveMessage request will contain at least one of the available messages. 
            + Reduce the cost of using Amazon SQS by reducing the number of empty responsed and false empty responses. 


+ deleteMessage
    + When you receive the message, you **must delete it from the queue** to acknowledge that you processed the message and no longer need it. 
    + You specify which message to delete by providing the **receipt handle** that Amazon SQS returned when you received the message.
+ deleteMessageBatch
+ PuregeQueue
    + delete all the messages in an AmazonSQS queue without deleting the queue itself.  

### 2.3.2 Basic Queue Operations

+ CreateQueue 
    + attributes
        + delaySeconds
            + the delivery of all messages in the queue will be delayed 
            + default 0, maximum 15 min
        + maximumMessageSize
            + the limit of how many bytes a message can contain before Amazon SQS rejects it 
            + max 256 KB
        + messageRetentionPeriod
            + seconds SQS retains a message 
        + ReceiveMessageWaitTimeSeconds
            + time for which a ReceiveMessage call will wait for a message to arrive 
            + max configurable wait time is 20 seconds 
            + default 0
        + VisibilityTimeout
            + period of time that a message is invisbile to the rest of your application  
+ SetQueueAttributes
+ GetQueueAttributes
+ GetQueueUrl
+ ListQueues
+ DeleteQueue

## 2.4 Message Lifecycle

+ Immediately after a message is received, it **remains in the queue**. To prevent other consumers from processing the message again, Amazon SQS sets a visibility timeout, a period of time during which Amazon SQS prevents other consumers from receiving and processing the message. 
+ The default visibility timeout for a message is 30 seconds. The maximum is 12 hours.
+ Or consumer could send a separate request which acknowledges that you no longer need the message because you have successfully received and processed it 
+ Maximum message retention period
    + SQS automatically deletes messages that have been in a queue for more than maximum message retension period
    + default is 4 days 
    + can be set from 60 seconds to 14 days 

## 2.5 Queue and Message Identifiers 

+ Queue URL 
    + when creating a new queue, must provide a queue name that is unique within the scope of all your queues. 
    + AWS assgin each queue an identifier called a **queue URL**, which includes the queue name and other components that Amazon SQS determines. 
+ Message ID
    + For each message, Amazon SQS returns a system-assigned message ID in the SendMessage response. 
+ Receipt Handle
    + Each time you receive a message from a queue, you receive a receipt handle for that message. 
    + The handle is assoiciated with the act of receiving the message, not with the message itself. 
    + To delete a message, you need the message's receipt handle instead of the message ID. 

## 2.6 Dead letter queues

+ A queue of messages that were not able to be processed 
+ Use dead-letter queues with standard queues.
+ Dead letter queues help you troubleshoot incorrect message transmission operations 

## 2.7 Sharing a Queue

+ Shared queues
    + Queue can be shared with other AWS accounts
    + Queue can be shared anonymously 
    + A permission gives access to another person to use your queue in some particular way
    + A policy is the actual document that contains the permissions you granted 

## 2.8 Use cases 

+ Work queues
    + decouple components of a distributed application that may not all process the same amount of work simultaneourly 
+ Buffer and batch operations
    + add scalability and reliability to your architecture and smooth out temporary volume spikes without losing messages or increasing latency 
+ request offloading
    + move slow operations off of interactive request paths by enqueuing the request
+ Auto scaling
    + Use queue to help determine the load on an application, and when combined with auto sclaing, you can sclae the numebr of Amazon Ec3 intances out or in, depending on the volumne of traffic 
+ fan out
    + combine SQS with SNS to send identical copies of a message to multiple queues in parallel for simultaneous processing  

# 3. Amazon Simple Notification Service

## 3.1 Introduction 

+ A web service that makes it easy to set up, operate and send notifications from the cloud.
+ Follow the publish-subscribe messaging paradigm, with notifications being delivered to clients using a push mechanism that eliminates the need to periodically check or poll for new information and updates 
+ When using Amazon SNS, you (as the owner) create a **topic** and **control access to it by defining policies** that determine which publishers and subscribers can communicate with the topic. 
+ A publisher sends messages to topics they have created or to topics they have permission to publish to. 
+ Instead of including a specific destination address in each message, a publisher sends a message to the topic. 
+ Amazon SNS matches the topic to a list of subscribers who have subscribed to that topic and delivers the message to each of those subscribers. 
+ Each topic has a **unique name** that identifies the Amazon SNS endpoint for **publishers to post messages and subscribers to register for notifications**. Subscribers receive all messages published to the topics that they subscribe to, and all subscribers to a topic receive the same messages.
+ Subscriber
    + Web servers 
    + email addresses
    + amazon sqs queues
    + aws lambda
+ topic 
    + an access point for allowing recipients to dynamically subscribe for identical copies of the same notification 


## 3.2 Use case: Fan out 

+ An Amazon SNS message is sent to a topic and then replicated and pushed to multiple Amazon SQS queues, HTTP endpoints, or email addresses. 
+ Allow for parallel asynchronous processing
+ All subscribers get identical information 

## 3.3 Operations 

+ CreateTopic
    + Input: Topic name 
    + Output: ARN of topic 
    + creates a topic to which notifications can be published 
    + action is idempotent, so if the requester already owns a topic with the specified name, that topic's ARN is returned without creating a new topic 
+ Subscribe
    + Input
        + subscriber's endpoint
        + protocol 
        + ARN of topic 
    + prepare to subscribe an endpoint by sending the endpoint a confirmation message
    + to actually create a subscription, the endpoint owner must call the confirmSubscription action with the token from the confirmation message. 
    + The ConfirmSubscription request verify an endpoint owner's intent to receive messages by validating the token sent to the endpoint by an earlier Subscribe action. 
    + If the token is valid, the action creates a new subscription and returns its ARN 
+ DeleteTopic 
    + Input
        + ARN of topic  
    + Deleting a topic might prevent some messages previously sent to the topic being delivered to subscribers
    + Action is idempotent, will not result in an error if the topic doesn't exist
+ Publish 
    + Input 
        + Message
        + Messsage attributes
        + Message structure : json
        + subject 
        + ARN of topic 
    + output 
        + message ID 
    + Sends a message to all of a topic's subscribed endpoints.
    + When a messageId is returned, the message has been saved and Amazon SNS will attempt to deliver it to the topic's subscribers shortly. 
    + The format of the outgoing message to each subscribed endpoint depends on the notification protocol selected.

## 3.4 Best practices

### 3.4.1 Characteristics of Amazon SNS 

+ Each notification message contains a single published message
+ Message order is not guaranteed
+ A message cannot be deleted after it has been published 
+ Amazon SNS delivery policy can be used to control retries in case of message delivery failure
+ Message can contain up to 256 kb of text data

### 3.4.2 Manage Access to Amazon SNS

+ which endpoints
+ who can publish notifications
+ who can subscribe to notifications


### 3.4.3 SQS vs SNS 

+ both messaging services within AWS
+ SNS
    + allow applications to send time-critical messages to multiple subscribers through a push mechanism
    + eliminate the need to periodically check or poll for updates 
+ SQS 
    + message queue service used by distributed applications to exchange messages through a polling model, and it can be used to decouple sending and receiving components. 
    + provides flexibility for distributed components of applications to send and receive messages without requiring each component to be concurrently available
