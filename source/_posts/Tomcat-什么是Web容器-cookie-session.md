---
title: 'Tomcat - 什么是Web容器, cookie & session'
date: 2020-02-01 14:22:02
categories: Web
tags:
    - Tomcat
top:
---
# 1. 什么是Web容器

早期的Web应用主要用于浏览新闻等静态页面，HTTP服务器(比如Apache、Nginx)向浏览器返回静态HTML，浏览器负责解析HTML，将结果呈现给用户。 

而后我们希望能够在页面上有一些交互操作，获取动态的结果，因此就需要一些扩展机制来使得HTTP服务器能够调用服务端的程序。于是Sun公司推出了Servlet技术，其没有main方法，需要被部署到Servlet容器当中，由容器来实例化并调用Servlet。

Tomcat和Jetty就是HTTP服务器 + Servlet容器，又称为Web容器。

# 2. Cookie & Session


浏览器将请求打包成HTTP协议格式，当这个请求到达服务端的时候，会被Tomcat将HTTP请求数据字节流解析成一个Request对象，这个Request对象封装了HTTP所有的请求信息，接着Tomcat将这个请求交给Web应用去处理，处理完以后得到一个Response对象，Tomcat就会把这个Response对象转成HTTP格式的相应数据并发送给浏览器。

HTTP协议是无状态的，请求之间没有关系，为了让请求之间建立联系，设计出了Cookie还有Session技术。

## 2.1 Cookie技术

Cookie是HTTP报文的请求头，Web应用可以将用户的标识信息或者其他一些信息存储在Cookie中。用户通过验证之后，每次HTTP请求报文中都包含Cookie，这个服务器读取这个Cookie请求头就知道用户是谁了。

其本质上就是一份存储在用户本地的文件，里面包含了每次请求中都需要传递的信息。

## 2.2 Session技术

Cookie以明文的方式存储在本地，而Cookie中往往带有用户信息，Session就用来解决这个问题。是在服务端开辟的存储空间，里面保存了用户的状态。

用户信息以Session的形式存储在服务端，当用户请求到来时，服务端可以把用户的请求和用户的Session对应起来。

+ 服务器创建Session的时候，生成Session ID
+ 当浏览器再次发送请求时，在Cookie里会带上Session ID
+ 服务器根据SessionID找到对应的Session，并在Session当中获取或者添加内容