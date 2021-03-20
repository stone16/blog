---
title: Route53
date: 2021-03-20 13:10:21
categories: Cloud
tags:
    - DNS 
top:
---
# Route 53

# 1. How Route53 works?

- Intro
    - DNS webservice
    - functionalities
        - domain registration
        - DNS routing
        - health checking
- Domain Registration
    - You choose a domain name and confirm that it's available
    - You provide names and contact information for the domain owner and other contacts
    - Route 53 automatically makes itself the DNS service for the domain by doing:
        - create a hosted zone that has the same name as your domain
        - Assigns a set of **four name servers to the hosted zone**. When someone uses a browser to access your website, such as [www.example.com](http://www.example.com/), these name servers tell the browser where to find your resources, such as a web server or an Amazon S3 bucket.
        - Gets the name servers from the hosted zone and adds them to the domain.

# 2. Concepts

- Hosted Zone
    - A container for records
        - include info about how you want to route traffic for a domain and all of its subdomains
        - It has the same name as domain
        - 
- Records
    - Created in your hosted zone
    - For routing traffic to your resources
    - Each record includes information about how you want to route traffic for your domain
        - Name
        - Type
        - Value
- Name server
    - Route53 will assign a set of 4 name servers to the hosted zone
    - name server tell the accessor (browser) where to find your resources
- Domain Name System Concepts
    - Alias Record
        - Record you create to route traffic to AWS resources
    - subdomain
        - A domain name that has one or more labels prepended to the registered domain name. For example, if you register the domain name [example.com](http://example.com/), then [www.example.com](http://www.example.com/) is a subdomain. If you create the hosted zone [accounting.example.com](http://accounting.example.com/) for the [example.com](http://example.com/) domain, then [seattle.accounting.example.com](http://seattle.accounting.example.com/) is a subdomain.

# 3. Working with Hosted Zones

[https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html) 

# 4. Routing Traffic for subdomains

[https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html) 

- Option 1:
    - Create records in the hosted zone for the doamin
    - we could create a record named [test.example.com](http://test.example.com) in the example.com hosted zone
- Option 2:
    - Create a hosted zone for the subdomain, and create records in the new hosted zone

# 5. Routing Traffic to AWS Resources

[https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-aws-resources.html](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-to-aws-resources.html) 

- The logic is to leverage on AWS PrivateLink for cross vpc connection, create the interface endpoint in your service, and build connection between your VPC and Route53