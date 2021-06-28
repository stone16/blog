---
title: 'Log4j 是如何工作的？ '
date: 2021-04-24 18:15:48
categories: BackEnd

tags:
    - log4j
    - log 
top:
---
# Log4j 是如何工作的？

Created: Apr 24, 2021 5:02 PM
Tags: backend, tech
status: In Progress

# 1. Overview

- 组件化设计的日志系统

```
log.info("User signed in.");
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 ├──>│ Appender │───>│  Filter  │───>│  Layout  │───>│ Console  │
 │   └──────────┘    └──────────┘    └──────────┘    └──────────┘
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 ├──>│ Appender │───>│  Filter  │───>│  Layout  │───>│   File   │
 │   └──────────┘    └──────────┘    └──────────┘    └──────────┘
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 └──>│ Appender │───>│  Filter  │───>│  Layout  │───>│  Socket  │
     └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

- 通过appender将同一条日志输出到不同的目的地

## 1.1 Logger组件

- 负责产生日志，
- 级别
    - DEBUG
    - INFO
    - WARN
    - ERROR
    - FATAL

## 1.2 Appenders 组件

- 负责将日志输出到不同的地方
    - 控制台 Console
    - 文件 Files
        - 根据天数或者文件大小来产生新的文件

## 1.3 Layout

- 完整文档
    - [https://logging.apache.org/log4j/2.x/manual/layouts.html](https://logging.apache.org/log4j/2.x/manual/layouts.html)
- 说明你的日志要以何种格式来进行输出

```jsx
－X号: X信息输出时左对齐；

%p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL,

%d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}输出类似：2002年10月18日 22：10：28，921

%r: 输出自应用启动到输出该log信息耗费的毫秒数

%c: 输出日志信息所属的类目，通常就是所在类的全名

%t: 输出产生该日志事件的线程名

%l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)

%x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。

%%: 输出一个"%"字符

%F: 输出日志消息产生时所在的文件名称

%L: 输出代码中的行号

%m: 输出代码中指定的消息,产生的日志具体信息

%n: 输出一个回车换行符，Windows平台为"/r/n"，Unix平台为"/n"输出日志信息换行
```

# 2. XML 配置文件格式

- 通过Filter来过滤哪些需要输出，哪些不需要
- 通过Layout来格式化日志信息

```jsx
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
	<Properties>
        <!-- 定义日志格式 -->
		<Property name="log.pattern">%d{MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36}%n%msg%n%n</Property>
        <!-- 定义文件名变量 -->
		<Property name="file.err.filename">log/err.log</Property>
		<Property name="file.err.pattern">log/err.%i.log.gz</Property>
	</Properties>
    <!-- 定义Appender，即目的地 -->
	<Appenders>
        <!-- 定义输出到屏幕 -->
		<Console name="console" target="SYSTEM_OUT">
            <!-- 日志格式引用上面定义的log.pattern -->
			<PatternLayout pattern="${log.pattern}" />
		</Console>
        <!-- 定义输出到文件,文件名引用上面定义的file.err.filename -->
		<RollingFile name="err" bufferedIO="true" fileName="${file.err.filename}" filePattern="${file.err.pattern}">
			<PatternLayout pattern="${log.pattern}" />
			<Policies>
                <!-- 根据文件大小自动切割日志 -->
				<SizeBasedTriggeringPolicy size="1 MB" />
			</Policies>
            <!-- 保留最近10份 -->
			<DefaultRolloverStrategy max="10" />
		</RollingFile>
	</Appenders>
	<Loggers>
		<Root level="info">
            <!-- 对info级别的日志，输出到console -->
			<AppenderRef ref="console" level="info" />
            <!-- 对error级别的日志，输出到err，即上面定义的RollingFile -->
			<AppenderRef ref="err" level="error" />
		</Root>
	</Loggers>
</Configuration>
```

# Reference

1. [https://www.liaoxuefeng.com/wiki/1252599548343744/1264739436350112](https://www.liaoxuefeng.com/wiki/1252599548343744/1264739436350112) 
2. [https://blog.csdn.net/Mos_wen/article/details/50598967](https://blog.csdn.net/Mos_wen/article/details/50598967)