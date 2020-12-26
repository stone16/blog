---
title: 使用AWS EC2搭建Halo博客
date: 2020-12-26 09:45:40
categories: Web
tags: 
    - EC2
    - AWS
top:
---

# 1. EC2 设置

在完成了AWS注册之后，登录后台，在EC2的看板左侧点击Instances，选择Launch Instances，这时候会带你进入到选择AMI的界面，按照Halo的推荐是选择CentOS比较合适，不过亲测了下RHEL，CentOS都没有什么问题，按照自己的需要 (如果新账号的话，会有eligible free tier)，可以免费使用一年，使用其即可。

对于运行的EC2实例，我们还需要对VPC, Security Group. Elastic IP做配置，目的是为了能够在VPC之外(公网)能够访问HTTP, HTTPS端口，一般来说就是80,还有443. 这里的整个过程(troubleshooting)可以根据这篇官方博客来做。

[排查 EC2 实例的互联网网关连接问题](https://aws.amazon.com/cn/premiumsupport/knowledge-center/ec2-connect-internet-gateway/)

# 2. Halo基本设置

在根据第一部分的说明设置好服务器之后，我们可以ssh上服务器，然后开始做Halo的基本设置

详情可以看Halo安装的官方教程 — [在Linux服务器部署Hal](https://halo.run/archives/install-with-linux.html)o

- 几个值得注意的地方
    - JVM启动内存的分配
    - halo版本的更新
    - 端口的设置

# 3. 反向代理

使用Catty或者Nginx来做反向代理，完成https证书的申请，在你自己域名的服务商下设置dns，开始访问你自己的博客。

Halo域名的配置与访问

# 5. Troubleshooting

1. 服务器上启动了服务，port开了但是Public Ip还是无法访问到

这的错误很可能不在开启的服务(halo) 方面，而在于EC2防火墙 VPC等的配置，检查下端口是否都正常开启，根据第一部分的排查EC2互联网网关连接问题的文章一步步排查，基本上可以解决。

2. 在服务器上查看开启的端口，发现服务只开在IPV6上而没有在IPV4上开启

发现这个问题是发现在使用telnet -tlnp 指令的时候，发现Halo的进程确实开启了，但是是监听在tcp6 下，在Ipv4下没有端口监听。查询资料发现Java 网络模块现在是默认先检察当前操作系统是否支持IPv6， 如果支持，就会直接使用IPv6， 否则才会使用ipv4. [StackOverflow 上的问答](https://stackoverflow.com/questions/44718174/spring-boot-application-listens-over-ipv6-without-djava-net-preferipv4stack-tru) 如果想要设置先监听ipv4的话，我们可以在指令上加上

```jsx
-Djava.net.preferIPv4Stack=true
-Djava.net.preferIPv4Addresses

// 整个语句如下所示 (是在/etc/systemd/system/halo.service这个文件里做配置) 
ExecStart=/usr/bin/java -server -Xms256m -Xmx256m -jar YOUR_JAR_PATH -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Addresses
```

[Connect to an Amazon EC3 instance on HTTP or HTTPS ports](https://aws.amazon.com/premiumsupport/knowledge-center/connect-http-https-ec2/)