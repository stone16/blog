---
title: Java日志记录Tips
date: 2020-09-14 20:03:32
categories: BackEnd
tags:
top:
---
使用Java来记录日志有几个需要注意的地方的

+ 首先日志框架很多，不同的类库有可能会使用不同的日志框架，如何兼容是一个问题
+ 配置文件的复杂性


Java体系的日志框架有：
+ Logback
+ Log4j
+ Log4j2
+ commons-logging 
+ JDK 自带的Java.util.logging

如果不同的包使用不同的日志框架的话，那管理就会变得非常麻烦。为了解决这个问题，就有了SLF4J -- Simple Logging Facade For Java 

+ 提供了统一的日志门面API，实现了中立的日志记录API
+ 桥接功能
    + 可以将各种日志框架的API桥接到SLF4J API上。这样一来，即便你的程序试用了各种日志API记录日志，最终都可以桥接到Slf4j门面API上
+ 适配功能
    + 实现slf4j和实际日志框架的绑定
    + slf4j知识日志标准，还是需要一个实际的日志框架


下面一起梳理下常见的日志记录中的错误

# 1. 理解Logback配置，避免重复记录


    <?xml version="1.0" encoding="UTF-8" ?>
    <configuration>
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] [%-5level] [%logger{40}:%line] - %msg%n</pattern>
            </layout>
        </appender>
        <logger name="org.geekbang.time.commonmistakes.logging" level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </logger>
        <root level="INFO">
            <appender-ref ref="CONSOLE"/>
        </root>
    </configuration>

要注意在使用appender的时候，appender是如何挂载的，上述代码将appender挂载在了两个不同的地方，而且两个都定义在了root下，所以会造成重复。 


对于需要将不同的日志放到不同的文件的应用场景，可以通过设置Logger的additivity属性来实现这个操作

```
 <logger name="org.geekbang.time.commonmistakes.logging" level="DEBUG" additivity="false">        
    <appender-ref ref="FILE"/>    
 </logger>
 ```
