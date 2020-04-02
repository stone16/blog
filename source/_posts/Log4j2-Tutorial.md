---
title: 'Log4j2 Tutorial '
date: 2020-04-01 22:27:56
categories: BackEnd
tags:
    - log
top:
---
# 1. Log4j2 overview 

## 1.1 Why log? 
Logging is an important component of the development cycle. 
+ provides precise context about a run of the application 
+ once inserted into code, the generation of logging output requires no human intervention 
+ moreover, log output can be saved in persistent medium to be studied at a later time 

## 1.2 Why Log4j2? 

+ Designed to be usable as an audit logging framework , will not lose events while reconfiguring 
+ Contains asynchrounous loggers, which is 10 times faster than log4j 1.x and logback 
+ Garbage free for stand alone applications 
+ Use a plugin system that makes it easy to extend the framework by adding new appenders, filters, layouts, lookups, and pattern converters
+ Support for custom log levels 


## 1.3 Log4j Architecture 

![Log4jClasses.jpg](https://i.loli.net/2020/04/02/AsXomqJciUyPVKh.jpg) 

Applications using the Log4j 2 API will request **a Logger with a specific name from the LogManager**. The LogManager will **locate the appropriate LoggerContext** and then obtain the Logger from it. If the Logger must be created it will **be associated with the LoggerConfig** that contains either a) the same name as the Logger, b) the name of a parent package, or c) the root LoggerConfig. LoggerConfig objects are created from Logger declarations in the configuration. The LoggerConfig is associated with the Appenders that actually deliver the LogEvents.

+ Filter 
    + apply in different time point 
        + before control is passed to any loggerConfig
        +  after control is passed to a LoggerConfig but before calling any Appenders
        +  after control is passed to a LoggerConfig but before calling a specific Appender
        +  on each Appender
+ Appender 
    + selectively enable or disable logging requests
    + allow logging requests to print to multiple destinations 
+ Layout 
    + Used to customize the output format 
    + Associate a layout with an appender 

# 2. Migration from Log4j to Log4j2 
https://logging.apache.org/log4j/2.x/manual/migration.html 

Look at sample 1 in above link for how to set up basic configuration 

For some tips: 

+ The main package in version 1 is org.apache.log4j, in version 2 it is org.apache.logging.log4j
+ Calls to org.apache.log4j.Logger.getLogger() must be modified to org.apache.logging.log4j.LogManager.getLogger().
+ Calls to org.apache.log4j.Logger.getRootLogger() or org.apache.log4j.LogManager.getRootLogger() must be replaced with org.apache.logging.log4j.LogManager.getRootLogger().
+ Calls to org.apache.log4j.Logger.getLogger that accept a LoggerFactory must remove the org.apache.log4j.spi.LoggerFactory and use one of Log4j 2's other extension mechanisms.
+ Replace calls to org.apache.log4j.Logger.getEffectiveLevel() with org.apache.logging.log4j.Logger.getLevel().
+ Remove calls to org.apache.log4j.LogManager.shutdown(), they are not needed in version 2 because the Log4j Core now automatically adds a JVM shutdown hook on start up to perform any Core clean ups.
    + Starting in Log4j 2.1, you can specify a custom ShutdownCallbackRegistry to override the default JVM shutdown hook strategy.
    + Starting in Log4j 2.6, you can now use org.apache.logging.log4j.LogManager.shutdown() to initiate shutdown manually.
+ Calls to org.apache.log4j.Logger.setLevel() or similar methods are not supported in the API. Applications should remove these. Equivalent functionality is provided in the Log4j 2 implementation classes, see org.apache.logging.log4j.core.config.Configurator.setLevel(), but may leave the application susceptible to changes in Log4j 2 internals.
+ Where appropriate, applications should convert to use parameterized messages instead of String concatenation.
+ org.apache.log4j.MDC and org.apache.log4j.NDC have been replaced by the Thread Context.

# Reference
1. https://logging.apache.org/log4j/2.x/manual/migration.html 