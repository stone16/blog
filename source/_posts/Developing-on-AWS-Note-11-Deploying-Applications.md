---
title: Developing on AWS Note.11 Deploying Applications
date: 2020-01-28 23:22:38
categories: Cloud
tags:
    - AWS
    - CI/ CD
top:
---
# 1. DevOps 

+ Application is the application plus all of the associated infrastructure
+ includes
    + VPCs 
    + load balancers
    + auto scaling groups 
    + Amazon RDS databases
    + Amazon S3 bucket 
    + elastiCache servers
+ deploying in the cloud helps break down these traditional silos. Bugs due to different environment. 
+ practices
    + microservices
    + CI/ CD
    + Infrastructure as code 
+ Tools 
    + AWS Code Services 
+ Under a DevOps model, development and operations are no longer siloed. 
+ Sometimes, those two functions are merged into a single team where engineers work across the entire application lifecycle, from development and test to deployment to operations, and develop a range of skills not limited to a single function. 
+ Quality assurance and security teams could become more tightly integrated with development and operations throughout the application lifecycle. 


# 2. Release Processes Major Phases
Each steps can be automated without the entire release process being automated. 


+ Source
    + Check in source code such as .java files 
    + Code review
+ Build
    + Compile code 
    + style checkers
    + code metrics
    + create container images 
+ Test 
    + Integration tests with other systems
    + load testing
    + UI tests
    + penetration testing 
+ Deploy
    + Deployment to production environments 
+ Monitor
    + Monitor in production to quickly detect unusual activity or errors

# 3. Understanding CI & CD

+ Continuous Integration 
    + The practice of checking code and verifying each change with an automated build and test process
    + Require teams to write automated tests which can improve the quality of the software being released and reduce the time it takes to validate that the new version of software is good. 
    + Architecture for cloud Continuous Integration
        + Developer commits changes to the central repo. Pre-commit hooks should run and verify that the code meets specified requirements
        + A CI server pulls the changes from the central repo and builds the code
        + The CI server runs all required tests against the new branch or mainline change. 
        + The CI server returns a report to developer and stops the build job if a failure occurs. 
        + If the changes pass the required tests, the CI server builds the artifacts
        + The CI server pushes artifacts to the package builder
        + The **package builder** gets configuration information from the version control system 
        + The pacakge builder uses the configuration information and the artifacts to build the specified packages 
        + the packages are stored in a repository 
        + the repo uses a post-receive hook to deploy specific packages to staging. 
        + Do not fear rollbacks. Errors will happen, mistakes will be made, and the benefit of employing version control systems is that when appropriate, you can always revert to a previously working state and save yourself the time and effort of trying to debug 

+ Continuous Delivery
    + It extends continuous Integration to include testing out to production-like stages and running verification testing against those deployments. 
    + CD may extend all the way to a production deployment, but they have some form of manual intervention between a code check-in and when that code is available for customers to use
+ Continuous Deployment 
    + extends continuous delivery and is the automated release of software to customers from check in through to production without human intervention.  


# 4. How do you deploy all infrastructure along with your application? 

+ Infrastructure as code 
    + Define your AWS environment so that it can be created in a repeatable automated fashion 
    + stand up identical dev/ test environments on demand 
    + use the same code to create your production environment that you used to create your other environments

