---
title: HTTP API认证与授权(三) - Digest Access
date: 2020-01-31 18:55:56
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---

# 1. Intro

HTTP 摘要认证.这种方式可以在发送各种敏感信息之前先确认用户的身份。其会先给用户名密码加一个哈希函数；而HTTP basic方式与之相对的是直接使用了简单可逆的Base64编码技术而不是任何加密技术，使得整个过程非常不安全，除非是在HTTPS的条件下。 (Transport Layer Security)

划重点: 
+ MD5 加密哈希
+ 使用nonce(随机数) 避免重复攻击

# 2. 流程



+ 请求方将用户名口令和域做一个MD5哈希`MD5(username:realm:password)` 然后发给服务器
+ 问题是用户名口令不怎么变的话那这个字符串也会不改变
+ 因此在认证过程当中加入了nonce和qop

## 2.1 Client发起请求(无认证)

发生在Client直接进入一个需要认证的网页，此时并没有携带用户名和密码的信息

    GET /dir/index.html HTTP/1.0
    Host: localhost

## 2.2 Server返回错误信息401

401代表的是未认证，服务器会返回401信息，并且在Response Header里携带`WWW-Authenticate`域,含有认证的realm的信息，以及一个随机生成的nonce


    HTTP/1.0 401 Unauthorized
    Server: HTTPd/0.9
    Date: Sun, 10 Apr 2014 20:26:47 GMT
    WWW-Authenticate: Digest realm="testrealm@host.com",
                            qop="auth,auth-int",
                            nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                            opaque="5ccc069c403ebaf9f0171e9517f40e41"
    Content-Type: text/html
    Content-Length: 153
    
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8" />
        <title>Error</title>
      </head>
      <body>
        <h1>401 Unauthorized.</h1>
      </body>
    </html>

## 2.3 Client输入信息，发送新请求

    GET /dir/index.html HTTP/1.0
    Host: localhost
    Authorization: Digest username="Mufasa",
                         realm="testrealm@host.com",
                         nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                         uri="/dir/index.html",
                         qop=auth,
                         nc=00000001,
                         cnonce="0a4f113b",
                         response="6629fae49393a05397450978507c4ef1",
                         opaque="5ccc069c403ebaf9f0171e9517f40e41"

这里的计算方式如下:

    HA1 = MD5(username:realm:password)
    HA2 = MD5(method:digestURI)
    // 这一步可以有更多的值一起做hash，比如 server nonce, request counter, client nonce, qop (quality of protection code)
    response = MD5(HA1:nonce:HA2)

值得注意的是nonce需要隔一段时间就失效，request counter要累加，以此尽量使其更加安全

## 2.4 Server返回认证成功的信息

    HTTP/1.0 200 OK
    Server: HTTPd/0.9
    Date: Sun, 10 Apr 2005 20:27:03 GMT
    Content-Type: text/html
    Content-Length: 7984




# 3. 总结

摘要认证这个方式会比之前的方式要好一些，因为没有在网上传递用户的密码，而只是把密码的MD5传送过去，相对会比较安全，而且，其并不需要是否TLS/SSL的安全链接。但是，别看这个算法这么复杂，最后你可以发现，整个过程其实关键是用户的password，这个password如果不够得杂，其实是可以被暴力破解的，而且，整个过程是非常容易受到中间人攻击——比如一个中间人告诉客户端需要一个。

# Reference

1. https://en.wikipedia.org/wiki/Digest_access_authentication
2. https://tools.ietf.org/html/rfc2617
3. https://tools.ietf.org/html/rfc2069
