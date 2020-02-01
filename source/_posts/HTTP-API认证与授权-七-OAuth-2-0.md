---
title: HTTP API认证与授权(七) - OAuth 2.0
date: 2020-01-31 20:24:28
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---
# 1. Intro

在前面，我们可以看到，从Digest Access， 到AppID+HMAC，再到JWT，再到OAuth 1.0，这些个API认证都是要向Client发一个密钥（或是用密码）然后用HASH或是RSA来签HTTP的请求，这其中有个主要的原因是，以前的HTTP是明文传输，所以，在传输过程中很容易被篡改，于是才搞出来一套的安全签名机制，所以，这些认证方法都是可以在HTTP明文协议下使用的。

这种使用签名方式大家可以看到是比较复杂的，所以，对于开发者来说，也是很不友好的，在组织签名的那些HTTP报文的时候，各种，URLEncode和Base64，还要对Query的参数进行排序，然后有的方法还要层层签名，非常容易出错，另外，这种认证的安全粒度比较粗，授权也比较单一，对于有终端用户参与的移动端来说也有点不够。所以，在2012年的时候，OAuth 2.0 的 RFC 6749 正式放出。

OAuth 2.0依赖于TLS/SSL的链路加密技术（HTTPS），完全放弃了签名的方式，认证服务器再也不返回什么 token secret 的密钥了，所以，OAuth 2.0是完全不同于1.0 的，也是不兼容的。

两个主要Flow
+ Authorization Code Flow 
+ Client Credential Flow 

# 2. 流程

## 2.1 名词定义

+ Third party application: 第三方应用程序
+ HTTP Service: HTTP服务提供商
+ Resource Owner: 资源所有者
+ User Agent: 用户代理
+ Authorization server: 认证服务器
+ Resource server: 资源服务器 

## 2.2 general idea

在客户端和服务提供商之间设置一个授权层，客户端只能通过授权层来到服务提供商那里获取信息。整个流程变成用户用用户名密码登录客户端，用户给客户端带有特定权限的token。客户端登录授权层以后，服务提供商根据token的权限范围和有效期，向客户端开放用户存储的资料。

## 2.3 运行流程

![fig1.png](https://i.loli.net/2020/02/01/Wdat63GcDsKT9mf.png)

+ A: 用户打开客户端以后，客户端要求用户给予授权
+ B: 用户同意给客户端授权
+ C: 客户端使用得到的授权，向认证服务器申请令牌
+ D: 认证服务器对客户端进行认证以后，确认无误，同意发放令牌
+ E: 客户端使用令牌，向资源服务器申请获取资源
+ F: 资源服务器确认令牌无误，同意向客户端开放资源

# 3 客户端授权模式
客户端必须要获得用户的授权才能获得令牌，授权方式有以下几种：

+ 授权码模式 (authorization code)
+ 简化模式 (implicit)
+ 密码模式 (resource owner password credentials)
+ 客户端模式 (client credentials)

## 3.1 授权码模式

通过客户端的后台服务器，与服务提供商的认证服务器进行交流。

![fig2.png](https://i.loli.net/2020/02/01/BTpKrmRWw9Ju2aE.png)

+ A: 用户访问客户端，后者将前者导向认证服务器
+ B: 用户选择是否给予客户端授权
+ C: 假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
+ D: 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
+ E: 认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）


A步骤，客户端申请认证的URI，包含以下参数：
+ response_type: 表示授权类型 值固定为code
+ client_id
+ redirect_uri
+ scope
+ state


    GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
            &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com


C步骤，服务器回应客户端的URI，包含以下参数
+ code: 授权码 一般设为10分钟的有效时间 且只能使用一次
+ state: 如果客户端的请求中包含这个参数，认证服务器的回应也需要包含同样的


    HTTP/1.1 302 Found
    Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
              &state=xyz

D步骤，客户端向认证服务器申请令牌的HTTP请求当中，包含如下参数：

+ grant_type: 使用的授权模式 authrization_code
+ code: 上一步获得的授权码
+ redirect_uri: 重定向URI，与A中参数需一致
+ client_id: 表示客户端ID


    POST /token HTTP/1.1
    Host: server.example.com
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
    Content-Type: application/x-www-form-urlencoded
    
    grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

E步骤: 认证服务器发送的HTTP回复，包含以下参数
+ access_token: 表示访问令牌
+ token_type: 令牌类型
+ expires_in: 表示过期时间
+ refresh_token: 表示更新令牌，用来获取下一次的访问令牌
+ scope: 权限范围


     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }


## 3.2 简化模式

不通过第三方的应用程序的服务器，直接在浏览器当中向认证服务器申请令牌，跳过授权码这个步骤。所有步骤在浏览器当中完成，令牌对访问者可见，且客户端不需要认证。

![fig3.png](https://i.loli.net/2020/02/01/6zt3Sjg4iMYoQn8.png)

（A）客户端将用户导向认证服务器。

（B）用户决定是否给于客户端授权。

（C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。

（D）浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。

（E）资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。

（F）浏览器执行上一步获得的脚本，提取出令牌。

（G）浏览器将令牌发给客户端。


A步骤，客户端发出HTTP请求，包含以下参数: 
+ response_type：表示授权类型，此处的值固定为"token"，必选项。
+ client_id：表示客户端的ID，必选项。
+ redirect_uri：表示重定向的URI，可选项。
+ scope：表示权限范围，可选项。
+ state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值



    GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com
    
    
C步骤，认证服务器回应客户端的URL，包含：
+ access_token 
+ token_type
+ expires_in
+ scope
+ state


    HTTP/1.1 302 Found
     Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
               &state=xyz&token_type=example&expires_in=3600


# Reference 
1. https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html 