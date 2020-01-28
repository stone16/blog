---
title: 浏览器输入url以后都发生了什么
date: 2020-01-27 22:06:56
categories: FrontEnd
tags:
	- FrontEnd
	- Browser 
	- CDN 
	- network
	- TCP/ IP
top:
---
从输入一个网址开始，都调用了哪些服务，经历了哪些步骤，深度解析。以输入www.google.com 为例。


# 1. Client端
一般来说，这里的Client指用户，即browser浏览器。这里我们以输入google.com为例。
## 1.1 输入提示
浏览器会根据历史访问，书签等信息给出输入建议。

还会根据默认搜索引擎的搜索记录，去匹配最近的搜索记录。

## 1.2 url解析
如果是不合法的地址，会转给默认的搜索引擎,例如如果你正在使用chrome，可以在url输入框输入你想要搜索的内容，然后搜索引擎会根据关键字进行搜索。

HSTS列表 安全策略机制，强行使用https

## 1.3 DNS解析

域名通过DNS转化为ip地址，这个转化主要是为了人机交互的友好型。没有人喜欢记一堆数字来访问一个网站。DNS做的事情就是把你输入的www.google.com翻译成计算机可以理解的IP地址，类似于192.188.1.1这种样子。

### 1.3.1查询过程

在解析的过程中，浏览器会由近及远寻找是否有缓存信息，即存没存从域名到地址的映射，整个查询过程分为如下几步，值得注意的是一旦查询到，就会立刻返回，不会再继续执行下去了。

1. 查看浏览器内部缓存

浏览器内会会存有在一段时间内你曾经访问过的网站的域名地址的映射。

2. 系统缓存

操作系统的缓存。浏览器会发出system call， 去询问操作系统是否存有相应的映射。

3. 路由器缓存， ISP缓存

查询路由器的缓存。如果在路由器缓存中没有找到映射，就会去ISP(Internet Service Provider)处去寻找

4. 本地DNS服务器

5. 域名服务器  根域服务器  -> 顶级域名服务器

寻找方式类似于一个树状结构，从最底层的子叶开始向上遍历，不停向更高级的域名服务器发出请求。这个过程会不停发送携带有请求和IP地址的数据包，会经过在client和server之间的多个网路设备直到其到达正确的DNS服务器。


# 2 网络
找到了正确的IP地址以后就要开始建立连接了，建立连接的过程一般会使用TCP协议，通过三次握手建立连接。
## 2.1 TCP连接

会用TCP，建立连接。并在Client和Server之间传递数据包。


### 2.1.1 IP封装  socket

### 2.1.2 TCP 三次握手

1. Client 发出建立连接的请求。数据包携带有`SYN`。
2. 如果Server有开放的端口，可以接受并建立连接，那么server会返回`SYN` + `ACK`, 告诉Client我可以接受你的请求。
3. Client收到Server的回应，发送`ACK`给Server。 连接建立。

给一个知乎连接，[为什么是三次握手，不是两次或者四次？](https://www.zhihu.com/question/24853633)  非常有意思的例子。

### 2.1.3 TCP 四次挥手

1. Client发起中断请求，发送`FIN`到server
2. Server收到请求，可能数据还没有发完。这个时候不会关闭socket，而是回复`ACK`，告诉Client知道了
3. Client进入`Fin_Wait`状态，继续等待Server端的`FIN`报文。Server端发送完毕后，会向Client发送`FIN`
4. Client收到后就回复`ACK`，并关闭连接


# 3 Server

这里主要描述TCP连接建立和断开之间发生的一些事情。

TCP/IP是个协议组，是网络层和传输层的协议。Client首先建立一条与服务器的TCP连接（上文中的三次握手）。而后Client发送HTTP请求，这里为了获得页面，会发送一个GET请求给服务器。请求会包含浏览器ID，用户数据头，连接头（包含额外信息，比如是否需要保持TCP连接等），从cookie获取的数据等。

Server收到Client的Request，会将请求传递给Request Handler，去处理请求（从数据库查找数据，处理数据，构建Response）。构建完毕后会返回一个Response。值得注意的是这个Response里会含有状态信息： 

+ 1xx informational message only  —— 包含信息
+ 2xx success of some kind  ——成功信息
+ 3xx redirects the client to another URL  ——将Client转到其他URL
+ 4xx indicates an error on the client's part  ——Client端错误 
+ 5xx indicates an error on the server's part  ——Server端错误



# 4 页面渲染

浏览器根据Resonse返回数据，渲染出DOM树，将返回的数据呈现在页面上。


# Reference
https://github.com/sunyongjian/blog/issues/34