---
title: AWS-VPC
date: 2020-01-29 20:32:16
categories: Cloud
tags:
    - AWS
    - VPC
top:
---
# 1. What is a VPC? 

A virtual private cloud (VPC) is a **virtual network** dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. Amazon Virtual Private Cloud (Amazon VPC) enables you to launch AWS resources into a virtual network that you've defined. This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS. 

![fig1.jpg](https://i.loli.net/2020/01/30/iduxJISlrmp5h2s.jpg)

It allows you to provision a **logically isolated section** of the AWS cloud, where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment including: 

+ selection of your own IP address ranges
+ creation of subnets 
+ configuration of route tables and network gateways 
+ create a hardware Virtual Private Network(VPN) connection between your corporate datacenter and your VPC and leverage the AWS cloud as an extension of your corporate datacenter. 

![fig2.jpg](https://i.loli.net/2020/01/30/kjVFzSHfdcyUspB.jpg)

# 2. Protection through Security Groups and Control Lists 

We can leverage multiple layers of security, including security groups and network access control lists, to help control access to Amazon EC2 instances in each subnet. **Security groups** act as a firewall for the Amazon EC2 instances controlling both inbound and outbound traffic **at the instance level**. **Network Acess Control Lists**(NACL's) act as a firewall for associated subnets controlling both inbound and outbound traffic **at the subnet level**. 


 **Security Group**| **Network ACL**
---|---
Operates at the instance level (first layer of defense). | Operates at the subnet level (second layer of defense)
Is stateful. Return traffic is automatically allowed, regardless of any rules. | Is stateless. Return traffic must be explicitly allowed by rules.
Applies to an instance only if someone specifies the security group when launching the instance, or associates the security group with the instance later on.|Automatically applies to all instances in the subnets associated with it (backup layer of defense, so you don't have to rely on someone specifying the security group).

# 3. Primary Components needed to configure networking in a VPC

## 3.1 [Network Interfaces](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html) 

A logical networking component in a VPC that represents a virtual network card. Includes attributes as following: 

+ a primary private IPv4 address

+ one or more secondary private IPv4 addresses

+ one Elastic IP address per private IPv4 address

+ one public IPv4 address, which can be auto-assigned to the network interface for eth0 when you launch an instance

+ one or more IPv6 addresses

+ one or more security groups

+ a MAC address

+ a source/destination check flag

+ a description

You can create a network interface, attach it to an instance, detach it from an instance, and attach it to another instance. A network interface's attributes follow it as it is attached or detached from an instance and reattached to another instance. When you move a network interface from one instance to another, network traffic is redirected to the new instance.

## 3.2 [Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

A route table contains a set of rules, called routes, that are used to determine where network traffic is directed.

Each subnet in your VPC must be associated with a route table; the table controls the routing for the subnet. A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same route table.

## 3.3 [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

A VPC component that allows communication between instances in your VPC and the Internet. In theory, it is what the traffic between your VPC and the public internet flows through.

## 3.4 [Egress-Only Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/egress-only-internet-gateway.html)

An egress-only Internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows outbound communication over IPv6 from instances in your VPC to the Internet, and prevents the Internet from initiating an IPv6 connection with your instances.

## 3.5 [DHCP Options Sets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html)

Dynamoc Host Configuration Protocol provides a standard for passing configuraion information to hosts on a TCP/IP network. The options field of a DHCP message contains the configuration parameters. Some of those parameters are the domain name, domain name server, and the netbios-node-type.

## 3.6 [Domain Name System](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html)

Domain Name System (DNS) is a standard by which names used on the Internet are resolved to their corresponding IP addresses. A DNS hostname is a name that uniquely and absolutely names a computer; it's composed of a host name and a domain name. DNS servers resolve DNS hostnames to their corresponding IP addresses.

Public IPv4 addresses enable communication over the Internet, while private IPv4 addresses enable communication within the network of the instance (either EC2-Classic or a VPC).

## 3.7 [Elastic IP Addresses](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-eips.html)

An Elastic IP address is a static public IPv4 address designed for dynamic cloud computing. You can associate an Elastic IP address with any instance or network interface for any VPC in your account. 

With an Elastic IP address, **you can mask the failure of an instance by rapidly remapping the address to another instance in your VPC.** Note that the advantage of associating the Elastic IP address with the network interface instead of directly with the instance is that you can move all the attributes of the network interface from one instance to another in a single step.

## 3.8 [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)

Enables private connectivity to services hosted in AWS, from within your VPC without using an Internet Gateway, VPN, Network Address Translation (NAT) devices, or firewall proxies.

## 3.9 [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat.html)

A highly available, managed Network Address Translation (NAT) service for your resources in a private subnet to access the Internet. A NAT device forwards traffic from the instances in the private subnet to the internet or other AWS services, and then sends the response back to the instances. When traffic goes to the internet, the source IPv4 address is replaced with the NAT device’s address and similarly, when the response traffic goes to those instances, the NAT device translates the address back to those instances’ private IPv4 addresses.

## 3.10 [VPC Peering](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-peering.html)

A peering connection enables you to route traffic via private IP addresses between two peered VPCs.

A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them privately. Instances in either VPC can communicate with each other as if they are within the same network. You can create a VPC peering connection between your own VPCs, with a VPC in another AWS account, or with a VPC in a different AWS Region.

# 4. Securing VPCs

## 4.1 Security Groups

A firewall for associated Amazon EC2 instances, controlling both inbound and outbound traffic at the instance level

## 4.2 Network Access Control Lists 

A firewall for associated subnets, controlling both inbound and outbound traffic at the subnet level.

## 4.3 Flow Logs 

Capture information about the IP traffic going to and from network interfaces in your VPC.

## 4.4 Route Tables 

A set of rules, called routes, that are used to determine where network traffic is directed

The following diagram illustrates the layers of security provided by security groups and network ACLs. In this example, traffic from an Internet gateway is routed to the appropriate subnet using the routes in the routing table. The rules of the network ACL associated with the subnet control which traffic is allowed to the subnet. The rules of the security group associated with an instance control which traffic is allowed to the instance.

![fig3.png](https://i.loli.net/2020/01/30/aeb1wYkCq2c9QXr.png)

# 5. Default VPC

The default VPC is suitable for getting started quickly, and for launching public instances such as a blog or simple website. With a default VPC, you don't have to deal with VPC creation and configuration. You can immediately start launching Amazon EC2 instances into your default VPC. You can also use services such as Elastic Load Balancing, Amazon RDS, and Amazon EMR in your default VPC.

You can use a default VPC as you would use any other VPC by: 

+ Adding additional non-default subnets
+ Modifying the main route table 
+ Adding additional route tables 
+ Associating additional security groups 
+ Updating the rules of the default security group
+ Adding VPN connections
+ Adding more IPv4 CIDR blocks 


The following figure illustrates the key components that we set up for a default VPC. 

![fig4.png](https://i.loli.net/2020/01/30/ig6ZwzSrFmqojPk.png)