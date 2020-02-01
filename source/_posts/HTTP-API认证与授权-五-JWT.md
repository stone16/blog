---
title: HTTP API认证与授权(五) - JWT
date: 2020-01-31 18:58:09
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---
# 1. 为什么选择JWT？

JWT也是一种Message Authentication Code的方法，选择使用JWT的好处是它可以将认证的逻辑交给第三方的服务器。而认证服务器和应用服务器之间也不需要有任何的直接连接。这样子做的好处就是应用服务器可以变成完全无状态的服务器了，不需要去存储token。

# 2. 步骤

1. 用户使用用户名和口令到认证服务器请求认证
2. 认证服务器验证以后，以服务器端生成JWT Token

+ 认证服务器生成一个Secret Key
+ 对JWT Header和JWT Payload分别求Base64
+ 用秘钥对JWT签名

3. 将base64(header).base64(payload).signature作为JWT token返回客户端
4. 客户端使用JWT Token向应用服务器发送相关的请求。这个JWT Token就像一个临时用户权证一样。


当应用服务器收到请求之后：

1. 检查JWT Token, 确认签名正确
2. 因为只有认证服务器有这个用户的Secret Key，所以应用服务器要将其传给认证服务器
3. 认证服务器通过JWT Payload解出用户的抽象ID，然后通过抽象ID查到登录时生成的Secret Key，然后再检查签名
4. 认证服务器检查通过后，应用服务就可以认为这是合法请求了

我们可以看以，上面的这个过程，是在认证服务器上为用户动态生成 Secret Key的，应用服务在验签的时候，需要到认证服务器上去签，这个过程增加了一些网络调用，所以，JWT除了支持HMAC-SHA256的算法外，还支持RSA的非对称加密的算法。

使用RSA非对称算法，在认证服务器这边放一个私钥，在应用服务器那边放一个公钥，认证服务器使用私钥加密，应用服务器使用公钥解密，这样一来，就不需要应用服务器向认证服务器请求了，但是，RSA是一个很慢的算法，所以，虽然你省了网络调用，但是却费了CPU，尤其是Header和Payload比较长的时候。所以，一种比较好的玩法是，如果我们把header 和 payload简单地做SHA256，这会很快，然后，我们用RSA加密这个SHA256出来的字符串，这样一来，RSA算法就比较快了，而我们也做到了使用RSA签名的目的。

# 3. 技术细节

## 3.1 构成

JWT有三部分组成：
+ header
+ payload
+ signature

### 3.1.1 payload 

payload里可以包含任何信息的，没有给任何限制。值得注意的是token并没有进行编码，所以当token被拦截的时候，里面的信息是可以被看到的

### 3.1.2 header

payload的内容在接收方是通过检查签名来进行验证的，而签名会有很多种。header里面就携带有不同签名的元数据(metadata).

### 3.1.3 签名

这一部分是一个Message Authentication Code, JWT的签名只有在获取了payload, header,还有被给予秘钥以后才可以生成。是三者的组合。

签名有很多种类，比如HS256，以及RS256. 

#### 3.1.3.1 HS256

使用HS256， 我们会用到Header， payload以及密码， 然后我们将其组合起来做哈希。想要生成同样的哈希值，你必须保证三个信息你都有才可以的(header, payload, password)，否则不可能得到，而且也无法通过碰撞逐渐趋近结果，因为hs256可以保证哪怕仅仅改变了一个数字，最终出来的结果也会有将近一半是不一样的。

#### 3.1.3.1 RS256

使用RS256比使用HS256效率高很多，这是因为：
1. HS256可以被暴力破解的，如果输入的秘钥很简单
2. 其需要server和client端有提前沟通好的一样的密码。这就意味着如果我们要换密码，我们就要提前传输到每个网络节点当中供其使用。

而使用RS256，我们仍要生成一个MAC码，目标仍然是创建一个数字签名来证明JWT是有效的。在这种签名当中，我们将会将创建token和验证token的能力分开。

实现这种目的的方式就是创建两个key而不是一个。
+ 一个私钥，只被认证服务器拥有，只被用来生成JWT
+ 私钥只能用来生成，不能用来证明
+ 公钥，用在应用服务器端，来认证JWT的
+ 公钥不需要当成隐私。可以随意让人使用的


使用RSA算法，RSA使用RSA keys。RSA算法可以用一个key来加密，另一个key来解密。但是RSA算法有个问题就是运行的比较慢。

这里采用的方法就是先把header和payload拿过来一起做一个hash，然后我们用私钥来对其进行加密，通过这种方式我们就能够得到RS256 的signature。

到了接收端，做的事情就是：
1. 拿到header和payload，然后用SHA-256做哈希，拿到真实数据
2. 用公钥进行解码，并且拿到签名的signature
3. 然后接收端比较两个哈希值，看是否相同



## 3.2 为什么使用JWT？ 为了解决什么问题而创建？

JWT使得认证服务器和应用服务器可以成为两个不同的服务器。这样的好处是使得应用服务器可以运行的更快，认证的功能都可以集中到认证服务器当中，然后再整个应用里面去复用。

## 3.3 JWT 实例

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

分成了三个部分，第一部分是JWT Header:

    JWT Header: 
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

第二部分是JWT Payload:

    JWT Payload: eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
    
第三部分是JWT Signature:

    JWT Signature: 
    TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

这里我们做了Base64编码，原因是各个电脑对于String有不同的处理方式，比如UTF-8, ISO 8859-1等等。使用Base64编码就可以解决这个问题了。

Base64和Base64url基本上是一样的，Base64url比起Base64，对在url中的展示做了一些优化。

## 3.4 JWT的用户Session管理

一般常用的payload有
+ user identification 
+ session expiration

    {
        // 给出JWT的实体- 这里是我们的认证服务器
        "iss": "Identifier of our Authentication Server",
        // JWT的创建时间的时间戳
        "iat": 1504699136, 
        // 用户的id
        "sub": "github|353454354354353453",
        // token expiration
        "exp": 1504699256
    }



# Reference 

1. https://blog.angular-university.io/angular-jwt/
2. https://coolshell.cn/articles/19395.html