---
title: HTTP API认证与授权(六) - OAuth 1.0
date: 2020-01-31 20:23:15
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---
# 1. Intro  

API认证协议，主要是为了做委托授权的。应用场景，比如用户想用第三方服务商来打印照片，访问其云存储，但是不想把用户名密码给这家第三方公司。

在这个模型当中，client可以代表资源的拥有者去做一些事情，也就是说，OAuth需要不仅能够确认证实资源拥有者得到授权认证，并且需要能够识别出提出请求的客户端的身份。

为了让客户端去访问资源：
1. 得到资源拥有者的许可 - 终端用户授权给客户端去访问服务器资源的流程 
2. 去服务器访问资源 - 利用两个证书（用于识别客户端生成的请求和用于识别请求所代表的资源拥有者）来生成已认证的Http请求。

三个角色
+ User  照片所有者 - 用户
+ Consumer 第三方照片打印服务
+ Service Provider 照片存储服务

协议的三个阶段
+ Consumer 获取Request Token
+ Service Provider认证用户并且授权Consumer
+ Consumer获取Access Token调用API访问用户的照片


整个授权流程： 

+ Consumer（第三方照片打印服务）需要先上Service Provider获得开发的 Consumer Key 和 Consumer Secret
+ 当 User 访问 Consumer 时，Consumer 向 Service Provider 发起请求请求Request Token （需要对HTTP请求签名）
+ Service Provider 验明 Consumer 是注册过的第三方服务商后，返回 Request Token（oauth_token）和 Request Token Secret （oauth_token_secret）
+ Consumer 收到 Request Token 后，使用HTTP GET 请求把 User 切到 Service Provider 的认证页上（其中带上Request Token），让用户输入他的用户和口令。
+ Service Provider 认证 User 成功后，跳回 Consumer，并返回 Request Token （oauth_token）和 Verification Code（oauth_verifier）
+ 接下来就是签名请求，用Request Token 和 Verification Code 换取 Access Token （oauth_token）和 Access Token Secret (oauth_token_secret)
+ 最后使用Access Token访问用户授权访问的资源

# Reference 

1. https://blog.csdn.net/turkeyzhou/article/details/7628399
2. https://oauth.net/1/