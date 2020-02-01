---
title: 'HTTP API认证与授权(四) - App Secret Key, HMAC'
date: 2020-01-31 18:56:59
categories: Web
tags:
    - HTTP
    - Network
    - 认证授权
top:
---
# 1. HMAC

HMAC,指Hash based message authentication code。用哈希给消息来进行签名，因为我们怕消息在传递的过程中被修改，所以我们对消息进行一个MAC算法，得到一个摘要字串；接收方在收到了信息以后，会进行同样的运算，然后来比较这个MAC字符串。如果一致，则表示没有被修改过。

![fig1.png](https://i.loli.net/2020/02/01/8ewgJ24qbQYjT7I.png)

# 2. App Id & App Secret Key 
App ID和验证无关，只是用来区分，是谁来调用API的，就像我们每个人的身份证一样，只是用来标注不同的人，不是用来做身份认证的。与前面的不同之处是，这里，我们需要用App ID 来映射一个用于加密的密钥，这样一来，我们就可以在服务器端进行相关的管理，我们可以生成若干个密钥对（AppID, AppSecret），并可以有更细粒度的操作权限管理。

## 2.1 S3 API 请求范例

1. 把HTTP的请求（方法、URI、查询字串、头、签名头，body）打个包叫 CanonicalRequest，作个SHA-256的签名，然后再做一个base16的编码
2. 把上面的这个签名和签名算法 AWS4-HMAC-SHA256、时间戳、Scop，再打一个包，叫 StringToSign。
3. 准备签名，用 AWSSecretAccessKey来对日期签一个 DataKey，再用 DataKey 对要操作的Region签一个 DataRegionKey ，再对相关的服务签一个DataRegionServiceKey ，最后得到 SigningKey.
4. 用第三步的 SigningKey来对第二步的 StringToSign 签名。

![fig2.png](https://i.loli.net/2020/02/01/5I6qCO1NBF9aQm3.png)

这种认证的方式好处在于，AppID和AppSecretKey，是由服务器的系统开出的，所以，是可以被管理的，AWS的IAM就是相关的管理，其管理了用户、权限和其对应的AppID和AppSecretKey。但是不好的地方在于，这个东西没有标准 ，所以，各家的实现很不一致。


# Reference

1. https://en.wikipedia.org/wiki/HMAC