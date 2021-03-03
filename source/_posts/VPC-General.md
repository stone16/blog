---
title: VPC General
date: 2021-03-02 20:42:42
categories: Cloud
tags: - VPC
top:
---
# VPC General

# 1. Overview

- Enable you to launch AWS resources into a virtual network that you've defined
- VPC helps you to resemble a traditional network that you'd operate in your own data center

## 1.1 Concepts

- VPC
- Subnet
    - A range of IP addresses in the VPC
- Route Table
    - A set of rules, called routes, used to determine where network traffic is directed
- Internet Gateway
    - A gateway that you attach to your VPC to enable communication between resources in your VPC and the internet
- VPC endpoint
    - Enable you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway , NAT device, VPN connection, or AWS Direct Connect Connenction
- CIDR block
    - classless inter domain routing

# 2. How Amazon VPC Work?

## 2.1 VPC and Subnets

- Use public subnet for resources that must be connected to the internet
- Use private subnet for resources that won't be connected to the internet
- To protect the AWS resources in each subnet
    - you could use Security groups, network access control lists

## 2.2 Accessing the internet

- Both subnets are public, they have both private and public IP address
- They could access internet via Internet gateway

![1.png](https://i.loli.net/2021/03/03/zdUi1WXuODGofjK.png)

- NAT Device vs Internet Gateway
    - When using NAT, it's another layer of protection, but ultimately, you still need to use Internet Gateway
    - NAT  — network address translation device
        - Allow instance in your VPC to instantiate outbound connections to the internet but prevent unsolicited inbound connections from the internet
        - It maps multiple private IPv4 addresses to a single public IPv4 address
        - A NAT device has an Elastic IP address and is connected to the internet through an internet gateway
        - You can connect an instance in a private subnet to the internet through the NAT device, which **routes traffic from the instance to the internet gateway**, and routes any responses to the instance.

## 2.3 Accessing services through AWS PrivateLink

- AWS PrivateLink enables you to privately connect your VPC to supported AWS services, services hosted by other AWS accounts (VPC endpoint services), and supported AWS Marketplace partner services.
- To use AWS PrivateLink, create a VPC endpoint for a service in your VPC
    - this creates an elastic network interface in your subnet with a private IP address that serves as an entry point for the traffic destined to the service

        ![2.png](https://i.loli.net/2021/03/03/KdQbqJLR2Or1DnB.png)

# 3. Getting Started

- Create a VPC with a /16 CIDR block
    - has 65,536 private IP addresses
- attach an internet gateway to the VPC
- for instances in public subnet, you need to assign an Elastic IP address to the instance
- create a size /24 subnet in the VPC
- create a custom route table, associate it with subnet
    - to flow traffic between the subnet and the internet gateway

## 3.1 EG - VPC with a single public subnet

### 3.1.1 Basic Setting

- VPC with/16 CIDR
- subnet with /24 CIDR
- internet gateway
    - help connect the VPC to the internet and to other AWS services
- custom route table
    - enable instances in the subnet to use IPV4 to communicate with other instances in the VPC

    ![3.png](https://i.loli.net/2021/03/03/PulS8ZxvFVXQtcJ.png)

### 3.1.2 Security

- Security Groups
    - control inbound and outbound traffic for your instances
- Network ACLs
    - control inbound and outbound traffic for your subnets

See the recommended Security Group Setting and Network ACLs setting [here](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html) 

## 3.2 EG - VPC with Public and Private subnets (NAT)

### 3.2.1 Overview

- Scenario
    - A public facing web application
        - maintain back end servers that are not publicly accessible
        - database server in private subnet while the webserver in a public subnet
- Public subnet vs private subnet
    - instances in the public subnet can send outbound traffic directly to the internet, whereas the instances in the private subnet can't
    - the instances in the private subnet can access the Internet by using a network address translation (NAT) gateway that resides in the public subnet
    - The database servers can connect to the Internet for software updates using the NAT gateway, but the Internet cannot establish connections to the database servers

    ![4.png](https://i.loli.net/2021/03/03/cb6tYu79jkWyMOH.png)

    ### 3.2.2 Security Setting

    [VPC with public and private subnets (NAT)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)

    ## 3.3 Sharing Public Subnets and Private Subnets

    - still need to leverage on PrivateLinks, VPC Peering, etc.

    [Example: Sharing public subnets and private subnets](https://docs.aws.amazon.com/vpc/latest/userguide/example-vpc-share.html)

    - Consider this scenario where you want an account to be responsible for the infrastructure, including subnets, route tables, gateways, and CIDR ranges and other accounts that are in the same AWS Organization to use the subnets. A VPC owner (Account A) creates the routing infrastructure, including the VPCs, subnets, route tables, gateways, and network ACLs. Account D wants to create public facing applications. Account B and Account C want to create private applications that do not need to connect to the internet and should reside in private subnets. Account A can use AWS Resource Access Manager to create a Resource Share for the subnets and then share the subnets. Account A shares the public subnet with Account D and the private subnet with Account B, and Account C. Account B, Account C, and Account D can create resources in the subnets. Each account can only see the subnets that are shared with them, for example, Account D can only see the public subnet. Each of the accounts can control their resources, including instances, and security groups.

    [VPC sharing: A new approach to multiple accounts and VPC management | Amazon Web Services](https://aws.amazon.com/cn/blogs/networking-and-content-delivery/vpc-sharing-a-new-approach-to-multiple-accounts-and-vpc-management/)

    ## 3.4 Service using AWS PrivateLink and VPC Peering

    [Examples: Services using AWS PrivateLink and VPC peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peer-region-example.html)

    [PrivateLinks](https://www.notion.so/PrivateLinks-e8e5f6802544401299fc3be21b10ed06)