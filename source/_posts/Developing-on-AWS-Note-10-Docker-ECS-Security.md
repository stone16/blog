---
title: 'Developing on AWS Note.10 Docker, ECS, Security'
date: 2020-01-28 23:20:55
categories: Cloud
tags:
    - AWS
    - Docker
    - ECS
    - Security
top:
---

# 1. Why Containers? 
## 1.1 Benefits 
+ software development lifecycle
    + source code
    + code repository 
    + build env
    + artifact repository 
    + test environment
    + deployment env
+ Containers
    + a method of **operating system virtualization** that allow you to run an application and its dependencies in resource-isolated processes. Containers allow you to easily package an application's code, configurations, and dependencies into easy to use building blocks that deliver **environmental consistency, operational efficiency, developer productivity, and version control** 
    + help resolve problems
        + different application stacks
        + different hardware deployment environments
        + run applications across different envs
        + migration to different envs
    + benefits
        + allow you to easily package an application's code, configurations, and dependencies into easy to use building blocks that deliver environmental consistency, operational efficiency, developer productivity, and version control
+ docker containers
    + decouple applications from operating systems 
+ Containers vs VM 
    + Containers can run on any Linux system with appropriate kernel feature support and the Docker daemon present. This makes them extremely portable. Your laptop, your VM, your EC2 instance, and your bare metal server are all potential hosts.
    + The lack of a hypervisor requirement also results in almost no noticeable performance overhead. The processes are talking directly to the kernel and are largely unaware of their container silo. Most containers boot in just a couple of seconds.
    + Where VMs are isolated at the operating system level, containers are isolated at the kernel level. This means that several applications can run on a single host operating system, and yet still have their own file system, storage, RAM, libraries, essentially, their own "view" of the system. 

## 1.2 When to use Docker Containers

+ Distributed apps and microservices
    + breaking up monoliths 
    + service oriented architecture
+ Batch Jobs
    + short lived jobs
    + variety and flexibility 
    + elastic
+ CI/ CD pipelins
    + packaging your code in a Docker images
    + test
    + deploy in production 

## 1.3 Microservices architecture with Containers

Using Docker, all of the different tiers of your Web application architecture could be **constructed as independent Docker containers**. If you are deploying a microservices architecture, you could **implement each service as its own separate Docker container**. Such an approach has several advantages: 
+ You can distribute running services across instances on an Amazon EC2 fleet.
+ You can **run multiple services on a single Amazon EC2 instance within your fleet**. This allows maximizing the CPU and memory usage of your existing instances.
+ You can **run multiple different versions of a service simultaneously** - even on the same machine, assuming that they bind to different ports. This allows you to release breaking changes in services while retaining backward compatibility with applications that may not have yet been modified to work against the new version.

## 1.4 Docker Image Registry

+ Amazon Elastic Container Registry (Amazon ECR) is a fully-managed Docker container registry that makes it easy for you to store, manage, and deploy Docker container images. 
+ Amazon ECR can be used standalone and also has deep integration with Amazon ECS, simplifying your development to production workflow. 
+ Amazon ECR eliminates the need to operate your own container repositories or worry about scaling the underlying infrastructure. 
+ Amazon ECR hosts your images in a highly available and scalable architecture, allowing you to reliably deploy containers for your applications.

# 2. Amazon Container Services

+ You need a way to intelligently place your containers on the hosts that have the resources and that means you need to know the state of everything in your system.
+ Amazon Container Services
    + management 
        + deploy, schedule, scale,
        + Elastic Container Service (ECS)
        + Elastic Container Service for Kubernetes (EKS)
    + hosting 
        + Amazon EC2
        + AWS Fargate
    + Image Registry 
        + Amazon Elastic Container Registry (ECR)

# 3. Developing Secure Applications 

## 3.1 AWS Certificates Manager 

+ AWS Certificate Manager is a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/ Transport Layer Security (SSL/ TLS) certificates for use with AWS services and your internal connected resources. 
+ SSL/ TLS are used to secure network communications and establish the identity of websites over the internet as well as resources on private networks. 
+ With ACM, you can quickly request a certificate, deploy it on ACM-integrated AWS resources, such as Elastic Load Balancing, Amazon CloudFront distributions, and APIs on API Gateway, and let ACM handle certificate renewals. 
+ It also enables you to create private certificates for your internal resources and manage the certificate lifecycle centrally. Public and private certificates provisioned through ACM for use with ACM-integrated services are free.

## 3.2 AWS Secrets Manager

+ Rotate, manage, and retrieve databse credentials, API keys, and other secrets throughout their lifecycle. 
+ IT administrators
    + store and manage access to secrets securely and at scale
+ Security administrators
    + audit and monitor the use of secrets, and rotate secrets without a risk of breaking applications 
    + offer secret rotation with build-in integration for Amazon RDS for MySQL, PostgreSQL and Amazon Aurora. 
+ Developers
    + avoid dealing with secrets in the applications 

## 3.3 AWS Security Token Service

+ provides trusted users with temporary security credentials 
+ configurable credential lifetime
+ once expired, cannot be reused
+ use IAM policies to control the privileges 
+ no limit on the number of temporary credentials issued 
+ Important points
    + all calls go to the global endpoint, by default
    + global endpoint maps to the US East region 
    + Regional endpoints are activated by default
    + use AWS cloudTrail to log SRS calls

## 3.4 Identity Providers
+ an alternative to create IAM users in AWS account
+ can manage user identities outside of AWS, and you can give these external user identities permissions to use AWS resources in account 
+ To use an identity provider, create an IAM identity provider entity to establish trust between your AWS account and the external identity provider.

## 3.5 Security Assetion Markup Language (SAML)

+ Use single sign-on to sign in to all of your SAML-enbabled applications by using a single set of credentials 
+ Manage access to your applications centrally 

## 3.6 Amazon Cognito

+ Allow saving and synchronizing user data on different devices
+ Allow access to AWS cloud services using 
    + public login providers
    + own user identity system 