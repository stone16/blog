---
title: HTTP调用的超时，并发
date: 2020-08-23 14:43:11
categories: Web
tags:
    - HTTP
top:
---

HTTP调用的时候，是通过HTTP协议进行一次网络请求，网络请求会有超时的可能性，我们需要考虑到：

+ 使用的框架设置的默认超时的合理性
+ 超时后的请求重试需要考虑到服务端接口的幂等性 -- 即任意多次执行所产生的影响是否与一次执行的影响相同
+ 需要考虑框架是否会限制并发连接数，以免在服务并发很大的情况下，HTTP调用的并发数限制成为瓶颈 

常用框架： 
+ Spring Cloud 
    + 需要使用Feign进行声明式的服务调用
+ Spring Boot
    + 使用Apache HttpClient进行服务调用
    
# 1. 如何配置连接超时

+ HTTP调用应用层走的是HTTP协议，但是网络层还是TCP/IP协议的
    + TCP/ IP协议是面向连接的协议，在传输数据之前需要建立连接
    + 网络框架会提供两个超时参数
        + 连接超时参数 ConnectTimeout
            + 建立连接阶段的最长等待时间
            + 应该配置在1 - 5s之间，因为TCP的三次握手建立连接需要的时间实际上是非常短的，超出往往是网络或者防火墙配置的问题

        + 读取超时参数 ReadTimeout
            + 用来控制从Socket上读取数据的最长等待时间
            + 读取超时包括
                + 网络问题
                + 服务端处理业务逻辑的时间

            + 参数配置不应过大
                + HTTP请求一般是同步调用，如果超时很长，在等待服务端返回数据的同时，客户端线程也在等待
                + 当下游服务出现大量超时的时候，程序可能也会受到拖累创建大量线程，最终崩溃


+ 首先对于超时本身
    + 是客户端和服务端需要都有贡献的
    + 有一致的时间估计
    + 平衡吞吐量和错误率


# 2. HTTP调用并发问题

如果使用Apache 的httpClient，在PoolingHttpClientConnectionManager当中，定义的参数： 

+ defaultMaxPerRoute = 2
    + 同一个主机最大的并发请求书为2
+ maxTotal = 20
    + 主机的最大并发为20

```

httpClient2 = HttpClients.custom().setMaxConnPerRoute(10).setMaxConnTotal(20).build();
```
