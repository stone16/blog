---
title: Tomcat - Servlet规范和Servlet容器
date: 2020-02-01 14:22:47
categories: Web
tags:
    - Tomcat
top:
---
# 1. Servlet规范

浏览器给服务端一个HTTP格式的请求，HTTP服务器收到这个请求之后，需要调用服务端程序来做处理。

为了解决耦合问题 -> 采用面向接口编程，就定义了一个接口，各种业务类都必须实现这个接口，这个接口就叫Servlet接口。

为了实例化Servlet，出现了Servlet容器，Servlet容器用来加载和管理业务类。

HTTP服务器并不直接跟业务类打交道，而是将请求交给Servlet容器去处理，Servlet容器会将请求转发到具体的Servlet，如果这个Servlet还没有创建，就去加载并且实例化这个Servlet，然后调用这个Servlet的接口方法。

Servlet接口其实是Servlet容器跟具体业务类之间的接口。

![fig1.jpg](https://i.loli.net/2020/02/02/Gfnmrp9AFldWq1s.jpg)

Servlet接口和Servlet容器这一整套规范就叫做Servlet规范。 Tomcat和Jetty都按照Servlet规范的要求实现了Servlet容器，同时它们也具有HTTP服务器的功能。

作为开发者，我们只需要实现一个Servlet，并将其注册到容器当中，剩下的事情就交由Tomcat来帮助我们解决了。

# 2. Servlet接口

Servlet接口定义了以下的方法：

    public interface Servlet {
        void init(ServletConfig config) throws ServletException;
        
        ServletConfig getServletConfig();
        
        void service(ServletRequest req, ServletResponse res）throws ServletException, IOException;
        
        String getServletInfo();
        
        void destroy();
    }


最重要的是Service方法，具体业务类在这个方法里实现处理逻辑。

+ 参数 - 本质上这两个参数是对通信协议的封装
    + ServletRequest 
        + 封装请求信息 
        + 其中包含所有请求的相关信息
            + 请求路径
            + Cookie
            + HTTP头
            + 请求参数
            + Session
    + ServletResponse
        + 封装响应信息 
+ Servlet容器在加载Servlet类的时候会调用init方法，卸载的时候会调用destroy方法。
+ ServletConfig的作用是封装Servlet的初始化参数，可以在web.xml中给Servlet配置参数，并在程序里通过getServletConfig方法拿到这些参数

# 3. Servlet容器

## 3.1 Servlet容器工作流程

+ 客户请求某个资源的时候，
+ HTTP服务器会用一个ServletRequest对象将客户的请求信息封装起来
+ 调用Servlet容器的Service方法
+ Servlet容器接到请求
+ 根据请求的URL和Servlet的映射关系，找到响应的Servlet
+ 如果Servlet还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化，接着调用Servlet的service方法处理请求
+ 将ServletResponse对象返回给HTTP服务器
+ HTTP服务器将响应发送给客户端

![fig2.jpg](https://i.loli.net/2020/02/02/jvc2pThotnA8J59.jpg)

## 3.2 Web应用的目录格式

Servlet是以Web应用程式的方式来进行部署的，而根据Servlet规范，WEB应用程序需要有一定的目录结构，在这个目录下放置了：

+ Servlet的类文件
+ 配置文件
+ 静态资源

    | -  MyWebApp
          | -  WEB-INF/web.xml        -- 配置文件，用来配置 Servlet 等
          | -  WEB-INF/lib/           -- 存放 Web 应用所需各种 JAR 包
          | -  WEB-INF/classes/       -- 存放你的应用类，比如 Servlet 类
          | -  META-INF/              -- 目录存放工程的一些信息


Servlet容器通过读取配置文件，就能找到并加载Servlet。

ServletContext这个接口用来对应一个Web应用，web应用部署好之后，Servlet容器在启动时会加载web应用，并为每个Web应用创建唯一的ServletContext对象。

ServletContext是个全局对象，一个Web应用可能有多个Servlet，这些Servlet可以通过全局的ServletContext来共享数据，包括Web应用的初始化参数，应用目录下的文件资源等

## 3.3 如何扩展和定制化Servlet容器的功能

Servlet规范提供了两中扩展机制，Filter and Listener

### 3.3.1 Filter 

Filter干预过程，是过程的一部分，是基于过程来被触发的

过滤器。允许你对请求和响应做一些统一的定制化处理，比如你可以根据请求的频率限制访问，或者根据不同国家修改响应内容。

+ WEB应用完成部署
+ Servlet容器实例化Filter
+ Filter会被链接成一个FilterChain
+ 当请求进来，获取第一个Filter并调用doFilter方法
+ doFilter方法负责调用这个FilterChain中的下一个Filter


### 3.3.2 Listener

Listener是基于状态的，任何行为改变同一个状态，触发的事件是一致的。

+ Servlet容器提供一些默认的监听器来监听事件
    + web应用的启动停止
    + 用户请求到达等
+ 事件发生，Servlet调动监听器的方法
+ 可以定义自己的监听器去监听你感兴趣的事件
+ 将监听器配置在web.xml中