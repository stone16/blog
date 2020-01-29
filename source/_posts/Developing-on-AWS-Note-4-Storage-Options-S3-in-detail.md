---
title: 'Developing on AWS Note.4 Storage Options, S3 in detail'
date: 2020-01-28 23:03:35
categories: Cloud
tags:
    - AWS
    - S3
top:
---

# 1. AWS Storage Options 

+ Amazon S3
    + Scalable, highly durable object storage in the cloud
+ Amazon Glacier
    + Low-cost, highly durable **archive** storage in the cloud
+ Amazon EFS
    + Scalable network file storage for Amazon EC2 instances
    + network file system that can grow to petabytes 
    + allows massively parallel access from EC2 instances to your data within a region
    + designed to **meet the performance needs of big data and analytics**
+ Amazon EBS
    + Network attached volumes that provide durable block-level storage for Amazon EC2 instances 
+ AWS storage gateway
    + change to hybrid later 
    + connects an on-premises software appliance with cloud-based storage to provide seamless and secure storage integration between an organization’s on-premises IT environment and the AWS storage infrastructure like Amazon S3, Amazon Glacier and EBS.

# 2. Amazon S3 

Amazon Simple Storage Service provides develipers with **high secure, durable, and scalable object storage**. 

## 2.1 Use cases

+ storage solution 
    + content storage and distribution
+ backup
+ archiving 
+ big data analytics
+ static web site hosting
+ disaster recovery 
    + cross region replication(CRR) automatically replicates every S3 object to a destination bucket located in a different AWS Region. 

## 2.2 Components 

+ bucket
    + global unique
    + use only lower case letters, numbers and hyphens
    + associated with a region
        + choose region by considering
            + latency
            + cost
            + regulatory requirements 
+ Object
    + S3 refers to files as objects 
    + you can store any number of objects inside bucket
    + each object is **identified by a unique key**
    + object metadata
    + version
        + each object has a version ID if you enable this feature
        + Object locking supported on versioned buckets
            + use object lock to prevent data from being changed, overwritten, or deleted 
    + URLs for S3 Objects
        + Path style URL
            + `http://<region-specific endpoint>/<bucket name>/<object name>`
        + virtual hosted-style URL
            + `http://<bucket name>.s3.amazonaws.com/<object key>` 
+ Key
    + unique identifier for each object in an S3 bucket 
+ Object Url
    + specify region, bucket name, object name(key)

## 2.3 Operations 

### 2.3.1 put 

+ upload object
+ copy object 
    + create copies of an object 
    + rename obejcts by creating a copy and deleting the original object 
    + move objects across S3 locations 
    + update object metadata 
+ limits
    + 5 GB at most in a single PUT operation 
    + recommened: use multipart upload if size > 100MB
        + Multipart upload allows you to upload a single object as a set of parts. 
        + You can upload each part separately. 
        + If one of the parts fails to upload, you can retransmit that particular part without retransmitting the remaining parts. After all the parts of your object are uploaded to the server, you must send a complete multipart upload request that indicates that multipart upload has been completed. 
        + Amazon S3 then assembles these parts and creates the complete object. 
        + Amazon S3 retains all parts on the server until you complete or abort the upload.
        + You can upload parts in parallel to improve throughput, recover quickly from network issues, pause and resume object uploads 

### 2.3.2 Get 
+ retrieve a complete object in a single GET request 
+ You can also retrieve an object in parts by specifying the range of bytes needed. This is useful in scenarios where network connectivity is poor or your application can or must process only subsets of object data.


### 2.3.3 Select 

+ Select content from Object instead of retrieving Object 
+ filter of content handled at S3 service level 
    + works by providing the ability to retrieve a subset of data from an object in Amazon S3 using simple SQL expressions 
    + simply change API from get to select 

### 2.3.4 Delete 

+ can delete a single object or delete multiple objects in a single delete request 
    + versioning disabled 
        + can permanently delete an object by specifying the key that you want to delete
    + versioning enabled 
        + can permanently delete an object by invoking a delete request with a key and version ID
        + must delete each individual version to completely remove an object 

### 2.3.5 Listing Keys 

+ There is no hierarchy of objects in S3 buckets. You can use prefixes in key names to group similar items. 
+ You can use delimiters (any string such as / or _) in key names to organize your keys and create a logical hierarchy

## 2.4 Features 

### 2.4.1 Pre-Signed URLs

+ Provide access to PUT/ GET objects without opening permissions to do anything else 
+ Use permissions of the user who creates the URL
+ Provide security credentials, a bucket name, an object key, HTTP method and expiration date and time 
+ onlu valid until expiration time 


### 2.4.2 Date Encryption 

+ Securing data in transit 
    + SSL-encrypted endpoints with HTTPS
    + client-side encryption - via SDKs
    + server-side encryption
        + S3 encrypts your data at the object level  
+ Securing data at rest on server
    + Amazon S3 managed keys (SSE-S3)
    + AWS KMS-managed keys (SSE-KMS)
    + Customer-provided keys (SSE-C)

### 2.4.3 Corss Origin Resource Sharing (CORS)

+ defines a way for client web applications that are loaded in one domain to interact with resources in a different domain.


## 2.5 Best practices 

+ Avoid unnecessary requests 
    + handle noSuchBucket errors instead of checking for existence of fixed buckets 
    + set the object metadata before uploading an object 
    + avoid using the copy operation to update metadata
    + cache bucket and key names if your application design allows it 
+ Network latency
    + choose the bucket region closest to latency-sensitive customers 
    + consider compressing data stored in Amazon S3 to reduce the size of data transferred and storage used
    + use a CDN to distribute content 
+ Data integrity
    + ensure the data has not been corrupted in transit
    + check MD5 checksum of the object retrieved from the GET and PUT operation 
        + AWS SDK automatically specifies MD5 checksum in a PUT operation. Amazon S3 recalculates MD5 checksum and compares it with the specified value. 
