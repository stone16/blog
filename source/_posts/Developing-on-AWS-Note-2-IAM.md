---
title: Developing on AWS Note.2  - IAM
date: 2020-01-28 23:00:14
categories: Cloud
tags:
    - AWS
    - Identity and Access Management
top:
---

# 1. Why need IAM? 
IAM: AWS Identity and Access Management
+ web service that helps you securely control access to AWS resources for your users. 
+ You use IAM to control **who** can use your AWS resources (**authentication**) and **what resources** they can use and in what ways (**authorization**).

+ To set up your dev env to wrok with the AWS SDK. 
    + Need an AWS acount and AWS credentials 
    + Use an IAM user to provide access credentials to ***increase the security of your AWS account***. 
+ grant least privilege - **grant only permissions required to perform a task**

# 2. Concepts 
![fig1.png](https://i.loli.net/2020/01/29/TbyX6r1jtKR2VuJ.png)
+ User
    + we can set up a user account for every developer in organization 
    + each user has credentials that they **must** use to access AWS services. 
+ groups 
+ roles 
    + trusted entities 
    + A role has policies granting access to specific services and operations 
    + create role like developer, and associate it with each developer user account
    + developer role can be configured with policies that control which services and operations that role has access to 
    + a role does not have standard long-term credentials(pwd or access keys) associated with it
+ Policy
    + contain permissions which specify which actions an entity can perform and on which resources 
    + a JSON document that defines effect, actions, resources, and optional conditions for what API calls an entity can invoke 
    + Type
        + Managed policy
            + standablon policies that you can attach to multiplke users, groups and roles 
            + reusability 
            + central change management 
            + version
            + rollback 
        + Inline policy
            + embedded in a principal entity like a user, group, or role. 
            + you can use the same policy across multiple entities, but those entities are not sharing the policy 
+ Resources 
    + The user, role, group, and policy objects that are stored in IAM
    + you can add, edit, and remove resources from IAM
+ Identities 
    + The IAM resource objects that are used to identify and group. These include users, groups, and roles.
+ Entities
    + The IAM resource objects that AWS uses for authentication
    + includes users and roles 
+ Principles
    + a person or application that uses an entity to sign in and make requests 
+ Authentication
    + As a principal, you must be authenticated (signed in to AWS) using an IAM entity to send a request to AWS.
    + Must provide your access key and secret key when accessing by CLI
+ Authorization 
    + Must be authorized to complete request 
    + AWS uses values from the request context to check for policies that apply to the request. 

# 3. Features 
+ management
    + user, role, federated users 
+ Shared access to your AWS account 
+ Granular permissions 
    + grant different permissions to different people for different resources
+ secire access tp AWS respirces for applications that run on Amazon EC2 
+ multi factor authentication 
+ eventually consistent 
+ identity based permissions
    + attached to the IAM user and indicate what the user is permitted to do.
    + attached to a resource and indicate what a specified user (or group of users) is permitted to do with it. **Amazon S3, Amazon Simple Queue Service (Amazon SQS), Amazon Simple Notification Service (Amazon SNS), and AWS OpsWorks are the only services that support resource-based permissions**.
+ resource based permissions
+ IAM Evaluation logic (In order)
    + By default, all requests are denied. (In general, requests made using the account/root credentials for resources in the account are always allowed.)
    + An explicit allow overrides this default.
    + An explicit deny overrides any allows

# 4. IAM best practice

+ IAM policies are specified with JSON-formatted text.
+ Policies are used to control access permissions for AWS APIs and other AWS resources. 
+ They are not used for operating system permissions or application permissions. For those, use LDAP or Active Directory/Active Directory Federation Services (AD FS).
+ When you create IAM policies, follow the standard security advice of granting least privilege; 
    + i.e., grant only the permissions required to perform a task. 
    + Determine what users need to do, and then craft policies for them that let the users perform only those tasks. 
    + Similarly, create policies for individual resources that identify precisely who is allowed to access the resource, and allow only the minimal permissions for those users. 

# 5. Amazon Shared Responsibility Model 

Customer and AWS table the responsibility together: 

+ customer
    + responsible for what you implement using AWS and for the applications you connect to AWS
+ AWS
    + goes from the ground up to the hypervisor. 
    + secure the hardware, software, facilities, and networks that run all of our products and services. Customers are responsible for securely configuring the services they sign up for and anything they put on those services. 