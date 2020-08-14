---
title: AWS CDK Overview
date: 2020-08-13 20:17:35
categories: Cloud
tags:
top:
---
# 1. Overview

+ AWS CDK 
    + Open source software development framework 
    + Model and provision your cloud application resources
    + To resolve what issue? 
        + provision cloud applications is challenging cause require
            + manual actions
            + custom scripts 
            + maintain templates 
            + domain specific languages 

    + How does CDK resolve the issue?
        + Provides with high level component that pre-configure cloud resources with proven defaults
        + Provision your resources through AWS CloudFormation 

## 1.1 Workflow 

+ Creating an Amazon ECS service with AWS Fargate launch type 

```
public class MyEcsConstructStack extends Stack {

    public MyEcsConstructStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyEcsConstructStack(final Construct scope, final String id,
            StackProps props) {
        super(scope, id, props);

        Vpc vpc = Vpc.Builder.create(this, "MyVpc").maxAzs(3).build();

        Cluster cluster = Cluster.Builder.create(this, "MyCluster")
                .vpc(vpc).build();

        ApplicationLoadBalancedFargateService.Builder.create(this, "MyFargateService")
                .cluster(cluster)
                .cpu(512)
                .desiredCount(6)
                .taskImageOptions(
                       ApplicationLoadBalancedTaskImageOptions.builder()
                               .image(ContainerImage
                                       .fromRegistry("amazon/amazon-ecs-sample"))
                               .build()).memoryLimitMiB(2048)
                .publicLoadBalancer(true).build();
    }
}
```

+ Basic workflow
    + create the app from a template provided by the AWS CDK
    + add code to the app to create resources within stacks
    + build the app 
    + synthesize one or more stacks in the app to create an AWS CloudFormation template 
    + deploy one or more stacks to your AWS account 

+ Benefits 
    + Could use logic when defining infrastructure 
    + Use object-oriented techniques to create a model of system 
    + Define high level abstractions

+ Tools
    + [CDK Toolkit](https://docs.aws.amazon.com/cdk/latest/guide/cli.html) 
        + CLI for interacting with CDK apps 
        + Enable developers to synthesize artifacts such as AWS CloudFormation templates, deploy stacks to development AWS accounts, and diff against a deployed stack to understand the impact of a code change 

    + [AWS Construct Library](https://docs.aws.amazon.com/cdk/latest/guide/constructs.html)
        + contains constructs representing AWS resources 
        + encapsulate the details of how to create resources for an Amazon or AWS service 

## 1.1.1 Create and build the app
```
mkdir hello-cdk && cd hello-cdk
cdk init TEMPLATE --language LANGUAGE 

cdk init app --language java
// In your IDE, import it as maven project 

mvn compile 

cdk ls 
```

### 1.1.2 Add an Amazon S3 Bucket 
```
// Add dependencies to pom.xml 
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>s3</artifactId>
    <version>${cdk.version}</version>
</dependency>

```
Define an Amazon S3 bucket in the stack using L2 construct 

```
package com.myorg;

import software.amazon.awscdk.core.*;
import software.amazon.awscdk.services.s3.Bucket;

public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
            .versioned(true).build();
    }
}
```

### 1.1.3 Systhesize an AWS CloudFormation Template 

```
cdk synth
```

### 1.1.4 Deploying the stack 

`cdk deploy`

### 1.1.5 Modifying the stack 

```
// after make your change 
cdk diff 
cdk deploy 

// Possibly destroy 
cdk destroy 

```

+ Synthesize before deploying 
# 2. Basic concepts
    
## 2.1 Constructs
### 2.1.1 Basics
+ Constructs 
    + Basic building blocks 
    + represents a cloud component, encapsulates everything AWS CloudFormation needs to create the component 
    + [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html
    + defferent level of constructs 
        + CFN Resources/ L1 
            + directly represent all of the AWS resources that are available in AWS CloudFormation 
            + named as CfnXyz, where xyz is name of the resource 
            + **When you use CFN resources, you must explicitly configure all resource properties, which requires a complete understanding of the details of the underlying AWS CloudFormation resource model**

        + AWS Reousources/ L2 Constrcuts
            + higher level, intent based API 
            + provide defaults, boilerplate, and glue logic you'd be writing with a CFN resource construct
            + Offer convenient defaults thus reduce the need for the detail of AWS resources  
        + Patterns - higher level constructs 
            + designed to help you complete common tasks in AWS, often involving multiple kinds of resources 

```
// L1 Construct
CfnBucket bucket = CfnBucket.Builder.create(this, "MyBucket")
                        .bucketName("MyBucket")
                        .corsConfiguration(new CfnBucket.CorsConfigurationProperty.Builder()
                            .corsRules(Arrays.asList(new CfnBucket.CorsRuleProperty.Builder()
                                .allowedOrigins(Arrays.asList("*"))
                                .allowedMethods(Arrays.asList("GET"))
                                .build()))
                            .build())
                        .build();

// L2 Construct
import software.amazon.awscdk.services.s3.*;

public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
                .versioned(true).build();
    }
}


```

### 2.1.2 Hierarchy - Composition
+ Composition 
    + High level construct can be composed from any number of lower level constructs 
    + In turn, those could be composed from even lower level constructs
    + Scoping pattern results in a hierarchy of constructs known as a construct tree 

+ Composition means you can define reusable components and share them like any other code 

### 2.1.3 Initialization

+ Being implemented in classes that extend the Construct base class 
+ 3 parameters 
    + scope 
        + the construct within which this construct is defined 

    + id 
        + an identifier that much be unique within this scope 
        + serves as a namespace for everything that's encapsulated within the scope's subtree 
        + used to allocate unique identities such as resource names and AWS CloudFormation logical IDs 

    + props 
        + a set of properties or keyword arguments 
        + define the construct's initial configuration 

# 3. Java Related 

## 3.1 AWS CDK idioms in Java 

+ Props 
    + expressed with Builder pattern 
    + define a bundle of key/ value pairs that the construct uses to configure the resources it creates 

```
Bucket bucket = new Bucket(this, "MyBucket", new BucketProps.Builder()
                           .versioned(true)
                           .encryption(BucketEncryption.KMS_MANAGED)
                           .build());
```

+ missing values 
    + it will be represented by `null`
    