---
title: JMX - Tutorials
date: 2020-02-08 21:15:44
categories: BackEnd
tags:
    - JMX
    - Java
top:
---
# 1. Introduction 

+ JMX - Java Management extensions
    + export standard metrics and custom metrics using MBeans to a monitoring system 
    + understand how your application is performing 
        + memory 
        + cpu
        + threads 
        + API calls in a REST endpoint

# 2. Why need this? 

+ Large scale java applications 
    + gather performance information 
        + number of users connected 
+ could provide a monitoring interface 
+ any class that exports data to JMX is called a Managed Bean(MBean). These MBeans publish their metrics to a MBean Server provided by the Java platform. 

# 3. Components 

## 3.1 MBean 

+ Objects with methods that return information and export the information via the MBeanServer
+ Mainly have 4 types 
    + Standard MBean 
        + create an interface with getter  
    + Dynamic MBean 
        + implements getters and setters to retrieve or modify the metric that can be auto discovered by implementing the javax.management.DynamicMBean interface 
    + Model MBean
        + Generic, dynamic in runtime to instrument the resources  
    + Open MBean
        + Using a predefined set of java classes

# 4. Example 

    // Create an interface that the MBeanServer will retrieve information 
    public interface SystemStatusMBean {
       Integer getNumberOfSecondsRunning();
       String getProgramName();
       Long getNumberOfUnixSecondsRunning();
       Boolean getSwitchStatus();
    }
    
Actual Implementation 

        public class SystemStatus implements SystemStatusMBean {
       private Integer numberOfSecondsRunning;
       private String programName;
       private Long numberOfUnixSecondsRunning;
       private Boolean switchStatus;
       private Thread backgroundThread;
    
       public SystemStatus(String programName) {
           // First we initialize all the metrics
           this.backgroundThread = new Thread();
           this.programName = programName;
           this.numberOfSecondsRunning = 0;
           this.numberOfUnixSecondsRunning = System.currentTimeMillis() / 1000L;
           this.switchStatus = false;
    
           // We will use a background thread to update the metrics
           this.backgroundThread = new Thread(() -> {
               try {
                   while (true) {
                       // Every second we update the metrics
                       numberOfSecondsRunning += 1;
                       numberOfUnixSecondsRunning += 1;
                       switchStatus = !switchStatus;
                       Thread.sleep(1000L);
                   }
               } catch (Exception e) {
                   e.printStackTrace();
               }
           });
           this.backgroundThread.setName("backgroundThread");
           this.backgroundThread.start();
       }
       
# 5. Operations and attributes  

Class properties exported through MBeans are called attributes, and methods exported through MBeans are called operations.

+ TotalCompilationTime 
    + Total time spent doing in JIT compilation  
+ Garbage Collector - CollectionCount 
    + Number of garbage collection events fired since the JVM launch
+ Garbage Collector - CollectionTime 
+ FreePyhsicalMemorySize 
+ CommitedVirtualMemorySize
    + The amount of memory that is guaranteed to be available for use by JVM  
+ ProcessCpuTime 
    + Time CPU has spent running the process  
+ PeakThreadCount 
    + maximum number of threads being executed at the same time since the JVM was started or the peak was reset  
+ ThreadCount 
    + the number of threads running at the current moment 

# Reference 
1. https://sysdig.com/blog/jmx-monitoring-custom-metrics/