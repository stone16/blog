---
title: SOLID - 里氏替换原则
date: 2020-03-13 21:33:36
categories: SystemDesign
tags: 
    - SOLID
top:
---
# 1. Intro
里氏替代原则 - Liskov Substitution Principle 

讲述的是子类对象需要能够替换程序当中父类对象出现的任何地方，并且保证原来程序的逻辑性为不变以及正确性不被破坏。


    public class Transporter {
      private HttpClient httpClient;
      
      public Transporter(HttpClient httpClient) {
        this.httpClient = httpClient;
      }
    
      public Response sendRequest(Request request) {
        // ...use httpClient to send request
      }
    }
    
    public class SecurityTransporter extends Transporter {
      private String appId;
      private String appToken;
    
      public SecurityTransporter(HttpClient httpClient, String appId, String appToken) {
        super(httpClient);
        this.appId = appId;
        this.appToken = appToken;
      }
    
      @Override
      public Response sendRequest(Request request) {
        if (StringUtils.isNotBlank(appId) && StringUtils.isNotBlank(appToken)) {
          request.addPayload("app-id", appId);
          request.addPayload("app-token", appToken);
        }
        return super.sendRequest(request);
      }
    }
    
    public class Demo {    
      public void demoFunction(Transporter transporter) {    
        Reuqest request = new Request();
        //...省略设置request中数据值的代码...
        Response response = transporter.sendRequest(request);
        //...省略其他逻辑...
      }
    }
    
    // 里式替换原则
    Demo demo = new Demo();
    demo.demofunction(new SecurityTransporter(/*省略参数*/););
    
# 2. 里氏替代原则 -- 按照协议进行设计

子类在设计的时候，应当遵守父类的行为约定。父类定义了函数的行为约定，那么子类可以改变函数的内部实现逻辑，但不能改变函数原有的行为约定。  

这里的行为约定指的是函数声明的要实现的功能；对于输入输出以及异常的约定

定义当中父类和子类之间的关系，也是可以替换成接口和实现类之间的关系的。