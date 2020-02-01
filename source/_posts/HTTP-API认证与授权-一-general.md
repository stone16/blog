---
title: HTTP API认证与授权(一) - general
date: 2020-01-31 18:52:42
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---
HTTP本身是无状态的，但我们常常需要检查用户的登录状态。一般来说，用户登录成功之后，服务器会发一个登录凭证(Token)。在计算机的世界当中，这个Token的相关数据会放到两个地方，一个在用户端，以Cookie的方式，另一个是放在服务器端，以Session的方式。

现实世界中验证登录会更为复杂一些，因为除了用户访问，还有用户委托的第三方的应用，还有企业之间的调用。

很多很有意思的问题： 
+ 认证与授权各自指的是什么？ 
+ 在我们没有TLS/ SSL的时候我们是如何实现登录验证，并且对在网络中传递的信息进行加密的呢？ 
+ HTTPS给整个认证与授权的过程带来了怎样的改变？ 
+ 我们常看到的通过微信/ google/ facebook/ amazon登录是如何实现的？ 
这一系列的博客会依次逐个解决上面描述的问题，inspired by [CoolShell-HTTP API 认证授权术](https://coolshell.cn/articles/19395.html). 是对这篇博客的针对自己现有认知水平（啥都不懂）的有效扩充，希望通过这一系列的整理彻底搞懂整个API认证与授权的机制，敏感信息在client和server之间的传递。

本系列的博客会大致上分为6部分，分别为：
+ HTTP Basic 
+ Digest Access
+ App Secret Key + HMAC
+ JWT
+ OAuth 1.0
+ OAuth 2.0 

希望能对大家有所裨益。


# Reference 

https://coolshell.cn/articles/19395.html