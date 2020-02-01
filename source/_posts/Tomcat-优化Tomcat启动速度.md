---
title: Tomcat - 优化Tomcat启动速度
date: 2020-02-01 14:20:22
categories: Web
tags:
    - Tomcat
top:
---
优化tomcat的启动速度，可以使得当你的service down掉，你做出修改的时候，能够更快的上线，这是个很可取，也很需要优化的方面了。

# 1. 清理Tomcat

## 1.1 清理不必要的web应用

删除掉webapps文件夹下不需要的工程：
+ host-manager 
+ example
+ doc

## 1.2 清理XML配置文件

Tomcat在启动的时候会解析所有的XML配置文件，但XML的解析的代价并不小，尽量保持XML配置文件的简洁。

## 1.3 清理JAR文件

可以删除所有不需要的JAR文件。JVM的类加载器在加载类时，需要查找每一个JAR文件来找到所需要的类，删除不需要的JAR文件，就可以使得查找的速度变快一些。

Web应用当中的lib目录下不应该出现Servlet API或者Tomcat自身的JAR，这些是由Tomcat负责提供的。

## 1.4 清理其他文件

及时清理日志，删除掉logs文件夹下不需要的日志文件。Catalina文件夹是Tomcat将JSP转换成Class文件的工作目录。每次启动会重新生成的。

# 2. 禁止Tomcat TLD扫描

TLD是对于标签库的定义，用来支持JSP的，如果你没有定义的话，那么就可以设置不去扫描这个JAR包 

+ 如果完全没有使用JSP作为页面模板，可以将TLD扫描禁掉


    <Context>
        <JarScanner>
            <JarScanFilter defaultTldScan = "false"/>
        </JarScanner>
    </Context>


+ 如果使用JSP作为模板，那么我们可以通过配置告诉Tomcat只扫描那些包括TLD文件的JAR包。找到`conf/`下的   ` catalina.properties  `文件，在这个文件里的jarsToSkip配置项当中，加入JAR包



    tomcat.util.scan.StandardJarScanFilter.jarsToSkip=xxx.jar

# 3. 关闭WebSocket的支持

Tomcat 会扫描 WebSocket 注解的 API 实现，比如@ServerEndpoint注解的类。我们知道，注解扫描一般是比较慢的，如果不需要使用 WebSockets 就可以关闭它。具体方法是，找到 Tomcat 的conf/目录下的context.xml文件，给 Context 标签加一个containerSciFilter的属性。

    <Context containerSciFilter="org.apache.jasper.servlet.JasperInitializer">
    </Context> 

# 4. 禁止Servlet注解的扫描

Tomcat会在web应用启动时扫描你的类文件，如果你没有使用servlet 注解，可以告诉Tomcat不要去扫描。具体配置方法：在你的 Web 应用的`web.xml`文件中，设置元素的属性`metadata-complete="true"`

# 5. 随机数熵源优化

Tomcat7以上版本依赖Java的SecureRandom类来生成随机数，比如SessionID, JVM默认使用阻塞式熵源(`/dev/random`), 某些情况下会导致tomcat启动变慢。

可以通过设置，让JVM使用非阻塞式的熵源。


     -Djava.security.egd=file:/dev/./urandom
     
# 6. 并行启动多个Web应用

Tomcat启动的时候，默认情况下Web应用时一个一个启动的，等所有Web 应用启动完成Tomcat才算启动完成。如果在一个Tomcat下我们有多个web应用，可以配置多个应用并行启动，通过修改server.xml文件当中的host元素的startStopThreads属性来完成。startStopThreads 的值表示你想用多少个线程来启动你的 Web 应用，如果设成 0 表示你要并行启动 Web 应用。

    <Engine startStopThreads="0">
        <Host startStopThreads="0">
        ...
        </Host>
    </Engine>
