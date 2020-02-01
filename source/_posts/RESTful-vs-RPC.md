---
title: RESTful vs RPC
date: 2020-01-31 20:58:01
categories: Web
tags:
    - RESTFul
    - RPC
top:
---
本文主要想分析二者的不同，以及为什么要采用RPC远程调用的方式，关于RPC协议本身，可以看这篇文章[RPC协议](https://www.llchen60.com/2018/12/15/RPC%E5%8D%8F%E8%AE%AE/)

# 1. REST详解

REST 代表 representational state transfer, 即表现层状态转移，划重点，状态的转移。 REST本身是不包含动作的，用一个个名词来划定资源，再通过定义的Get/Post/Put/delete等操作来做资源的交换。用Roy Fielding在其论文里的话来说： **REST is all about a client-server relationship, where server-side data are made available through representations of data in simple formats, often JSON and XML.** 
REST是来论述client和server之间的关系的，其中server端的数据是通过简单类型的数据(representation of data)来进行表示的，通常是JSON和XML.

这种数据的表现是可以进行修改的，我们可以通过方法以及多媒体(链接)来赋予动作和关系，然后进行各个状态的转移。相当于说，链接以及方法给予了状态转移的渠道和方式。

REST自身有一些限制：

+ REST是无状态的，请求之间没有持久的会话信息
+ 响应需要声明成可缓存的
+ REST关注一致性，如果使用HTTP，需要尽可能使用HTTP的特性，而不是去发明新的公约

REST之美体现在从任何状态向任何状态转移的合法的行为总是被server所控制的，与client的关系比较小；client是运行时候发出的请求。而对于RPC而言，它的行为会更加固定一些。对比二者之间的区别，你可以想象你通过不停点击链接从淘宝首页最终转到产品详情页的整个过程，和输入一个名词，通过一个API call直接到详情页的过程。

在上述第一个例子里，只要对server端的url链接做各种变化就可以了，让他能接受各种参数；但是对于第二种情况而言，我们需要在client端实现这个API call，然后从client向server发出一个请求，是需要在client和server端都进行修改的。

其实这里可以稍微加一点和RPC相比的“优势”，相对而言，RPC给工程师更多的操作空间，即你可以写出有着超强限定的API，但这样往往适用性会很低，然后随着时间，会出现N多API call，这对于后期的维护，开发都会造成不太好的影响。


# 2. What's RPC 

RPC，远程过程调用，前面写过一篇博文来讲它，大家可以[点击](https://www.llchen60.com/2018/12/15/RPC%E5%8D%8F%E8%AE%AE/)去看详情。这里说说它的发展的过程，起先的时候大家都用XML-RPC，奈何不怎么好用啊...因为XML对于各种类型的支持不是很好，大部分都只能当成String处理，这就尴尬了，你只能再附上metadata告诉别人这个到底是什么类型的。后面有了Json就好一些了。 

采用RPC比较难搞的还是你的API的精细度，解释一下，用REST的时候你可以通过query来询问不同的东西，这个时候实际上都是面对一个API的嘛，但是对于
RPC而言，我们是需要构建一个API专门来解决一个问题的。需要对其精细度（细化程度）做考量的。

# 3. 对比

其实二者没有孰优孰劣，甚至某种程度上这种比较是有点神奇的。因为实质上他们是可以相套的，比如从client看是个REST的API，但是从Server来看，在这个URL之下，实际上是用了RPC协议，从远程调用了另外一个服务的某个API，来使用。

REST更多的是面向状态，如果全都用REST来写API的话，你会发现他给出的几种预制的行为有些时候是无法很切实的描述出这个动作的；而RPC则可以很好的解决这一点，其主体就是个动作。


[1]. https://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/

[2] Roy Fielding's dissertation https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm 

[3]. https://www.smashingmagazine.com/2016/09/understanding-rest-and-rpc-for-http-apis/

[4]. https://zhuanlan.zhihu.com/p/34440779
