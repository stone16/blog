---
title: HTTP API认证与授权(二) - HTTP Basic
date: 2020-01-31 18:54:18
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---

# 1. HTTP Basic Intro

传统的API认证技术，使用username和password来进行登录。

整个流程是：
+ 用户发送一个不带认证信息的请求
+ 服务器返回401(unauthorized)状态，然后会在header里面包含一个`WWW-authenticate`域
+ 用户如果想认证自己，就需要发一个携带有认证请求header的请求，其中包含了credentials的信息。
+ 通常会让用户输入密码，然后将这些信息放到Authorization的header当中去

![fig1.png](https://i.loli.net/2020/02/01/NE6ihK9GQpjrw7P.png)

在上述的整个流程当中，因为相当于明文传输了，所以这整个过程必须发生在HTTPS(TLS)连接当中。
# 2.技术原理
+ 进行Base64编码
    + Base64编码是为了处理特殊字符，方便在不同平台用不同方式进行传递的，其编码方式是可逆的，即可以很顺畅地被破解掉。 
+ 将编码后的字段放到HTTP头的Authorization字段中，发到服务端
+ 服务端进行认证，成功则返回200；失败则返回401报错


使用Base64是为了消除特殊字符带来的影响，这种传递方式最大的问题是将用户名和口令放到了网络上进行传递，因此一般需要配合TLS/ SSL的安全加密来使用。这种同时将用户名和密码进行明文传输的协议并不是很好，尽管有HTTPS作为安全保护，但还是很有风险的。 


# Reference
1. https://en.wikipedia.org/wiki/Basic_access_authentication
2. https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication
