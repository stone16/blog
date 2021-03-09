---
title: 'Elastic Load Balancing '
date: 2021-03-08 20:59:23
categories: Cloud
tags:
    - Load Balancer 
top:
---
# Elastic Load Balancing

# 1. Overview

- What is ELB?
    - ELB distributes your incoming traffic across multiple targets, such as EC2 instances, containers, and IP addresses, in one or more AZs
    - It monitors the health of its registered targets, and routes traffic to the healthy targets
    - Elastic Load Balancing help scale your load balancer as your incoming traffic change over time — automatically scale to the vast majority of workloads

- Benefits
    - Increase the availability and fault tolerance
    - You could configure health checks, thus ELB monitor the health of the compute resources, LB only send requests to the healthy ones

- How it works
    - Listener
        - configure your LB to accept incoming traffic by specifying one or more listeners
        - a process that checks for connection requests
            - with a protocol
            - port number
    - cross zone load balancing
        - we should enable it cause it could make sure traffic is well distributed
        - when client send request, it will first go through route 53, and route 53 will distribute traffic thus each lb node receives 50% of traffic (2 LB nodes in total )
    - request routing
        - client — amazon dns service — return IP address to client side — client use the ip address to make call to LB
        - dns entry specify the TTL to 60 seconds, this ensure that the IP addresses can be remapped quickly in response to changing traffic

# 2. ELB Types

[https://aws.amazon.com/elasticloadbalancing/features/#Product_comparisons](https://aws.amazon.com/elasticloadbalancing/features/#Product_comparisons)  

# 3. Network Load Balancer

## 3.1 NLB Overview

- Similar to ELB overview, NLB has listener
    - a listener checks for connection requests from clients, using the protocol and port number you configure, and then forwards requests to a target group
- you can configure your health checks on a per target group basis
- Health checks are performed on all targets registered to a target group that is specified in a listener rule for your load balancer.
- functions at 4th layer of OSI, capable of handling millions of requests per second
    - when receives a connection request, it **selects a target from the target group** for the default rule
    - attempts to **open a TCP connection** to the selected target **on the port specified** in the listener configuration

## 3.2 How to create one NLB via Console?

- Create a target group
    - set target type, name, protocol, port number, health check method
- Configure load balancer and listener
    - Network mapping
        - select the VPC that you used for your EC2 instances
        - select AZ and then select public subnet for the AZ

## 3.3 Concepts

- Listener
    - A listener is a process that checks for connection requests, using the protocol and port that you configure. The rules that you define for a listener determine how the load balancer routes requests to the targets in one or more target groups.
- Target Groups
    - Each target group is used to route requests to one or more registered targets. When you create a listener, you specify a target group for its default action. Traffic is forwarded to the target group specified in the listener rule. You can create different target groups for different types of requests