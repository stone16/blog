---
title: Tomcat - 运行第一个 Servlet
date: 2020-02-01 14:19:17
categories: Web
tags:
    - Tomcat
top:
---
这篇文章会带着大家从下载安装Tomcat开始，编写自己的servlet，并且将其在Tomcat （Servlet 容器）当中进行运行，展现这整个过程。
 
 # 1. 安装Tomcat
 
 [官网链接](https://tomcat.apache.org/download-90.cgi)
 
 解压以后的目录结构如下：
 
 + bin
    + 存放在各个平台上启动和关闭Tomcat的脚本文件 
 + conf
    + 存放Tomcat的全局配置文件，其中最重要的是server.xml  
 + lib
    + 存放Tomcat以及所有Web应用都可以访问的JAR文件 
 + logs
    + 存放Tomcat执行时产生的日志文件 
 + work
    + 存放JSP编译后产生的Class文件 
 + webapps
    + Tomcat的web应用目录，默认情况下把Web应用放在这个目录下 

# 2. Servlet类编写

    import java.io.IOException;
    import java.io.PrintWriter;
    
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    
    public class MyServlet extends HttpServlet {
    
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
    
            System.out.println("MyServlet 在处理 get（）请求...");
            PrintWriter out = response.getWriter();
            response.setContentType("text/html;charset=utf-8");
            out.println("<strong>My Servlet!</strong><br>");
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response)
                throws ServletException, IOException {
    
            System.out.println("MyServlet 在处理 post（）请求...");
            PrintWriter out = response.getWriter();
            response.setContentType("text/html;charset=utf-8");
            out.println("<strong>My Servlet!</strong><br>");
        }
    
    }

将编写好的类编译成.class文件

    javac -cp ./servlet-api.jar MyServlet.java

# 3. 建立Web应用目录结构


    WebApp/WEB-INF/web.xml
    
    WebApp/WEB-INF/classes/MyServlet.class

然后在web.xml里面配置Servlet

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
      version="4.0"
      metadata-complete="true">
    
        <description> Servlet Example. </description>
        <display-name> MyServlet Example </display-name>
        <request-character-encoding>UTF-8</request-character-encoding>
    
        <servlet>
          <servlet-name>myServlet</servlet-name>
          <servlet-class>MyServlet</servlet-class>
        </servlet>
    
        <servlet-mapping>
          <servlet-name>myServlet</servlet-name>
          <url-pattern>/myservlet</url-pattern>
        </servlet-mapping>
    
    </web-app>

# 4. 运行

将WebApp放到Tomcat的安装目录下的webapps目录里

在bin目录下，启动脚本 
+ mac/ linux
    + startup.sh
+ windows
    + startup.bat


而后我们可以在浏览器看到结果

    http://localhost:8080/WebApp/myServlet

# 5. 查看日志

+ catalina.***.log
    + 主要记录Tomcat的启动过程
    + 可以看到启动的JVM参数以及操作系统等日志信息
+ catalina.out
    + 是Tomcat的标准输出和标准错误
+ localhost.***.log
    + 主要记录Web应用在初始化过程中遇到的未处理的异常，会被Tomcat捕获而输出到这个日志文件当中
+ localhost_access_log.***.txt
    + 存放访问Tomcat 请求的日志
    + 包括
        + IP地址
        + 请求路径
        + 请求时间
        + 请求协议
        + 状态码
+ manager.***.log/host-manager.***.log
    + 存放Tomcat自带的Manager项目的日志信息 