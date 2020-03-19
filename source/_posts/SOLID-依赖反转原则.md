---
title: SOLID - 依赖反转原则
date: 2020-03-18 20:53:05
categories: SystemDesign
tags:
    - SOLID
top:
---
# 1. 控制反转 IOC Inversion of Control 


    public class UserServiceTest {
      public static boolean doTest() {
        // ... 
      }
      
      public static void main(String[] args) {//这部分逻辑可以放到框架中
        if (doTest()) {
          System.out.println("Test succeed.");
        } else {
          System.out.println("Test failed.");
        }
      }
    }

上述代码程序员在自己控制整个代码的运行顺序和执行,可以看到如果我想增加test的话，就要在main函数里面添加，同时在UserServiceTest当中添加实例。而使用框架的话，代码就可以变成如下：


    public abstract class TestCase {
      public void run() {
        if (doTest()) {
          System.out.println("Test succeed.");
        } else {
          System.out.println("Test failed.");
        }
      }
      
      public abstract boolean doTest();
    }
    
    public class JunitApplication {
      private static final List<TestCase> testCases = new ArrayList<>();
      
      public static void register(TestCase testCase) {
        testCases.add(testCase);
      }
      
      public static final void main(String[] args) {
        for (TestCase case: testCases) {
          case.run();
        }
      }
      
上面的代码我们将测试的注册和运行都交给了JunitApplication了，然后我们要写新test，就让我们的concrete class extends TestCase类，来填充我们需要做的各种测试。

通过这种方式，我们实现了使用框架来控制整个代码的运转，我们使用框架来**组装对象，管理整个执行流程**。程序员利用框架进行开发的时候，只需要往预留的扩展点上，添加跟自己业务相关的代码，然后利用框架来驱动整个程序流程的执行。

控制反转是一种思想，有很多的具体的实现方式

# 2. 控制反转 -- 依赖注入

是控制反转的一种具体实现的方式，他的实际操作的指南是 -- 不通过new（）的方式在类内部创建依赖类的对象，而是将依赖的类对象在外部创建好以后，通过构造函数，函数参数的方式传递进来给类使用。


    // 非依赖注入实现方式
    public class Notification {
      private MessageSender messageSender;
      
      public Notification() {
        this.messageSender = new MessageSender(); //此处有点像hardcode
      }
      
      public void sendMessage(String cellphone, String message) {
        //...省略校验逻辑等...
        this.messageSender.send(cellphone, message);
      }
    }
    
    public class MessageSender {
      public void send(String cellphone, String message) {
        //....
      }
    }
    // 使用Notification
    Notification notification = new Notification();
    
    // 依赖注入的实现方式
    public class Notification {
      private MessageSender messageSender;
      
      // 通过构造函数将messageSender传递进来
      public Notification(MessageSender messageSender) {
        this.messageSender = messageSender;
      }
      
      public void sendMessage(String cellphone, String message) {
        //...省略校验逻辑等...
        this.messageSender.send(cellphone, message);
      }
    }
    //使用Notification
    MessageSender messageSender = new MessageSender();
    Notification notification = new Notification(messageSender);
    
使用依赖注入的最大好处就是，我们不需要在具体的类当中实例化其他的类，这样就实现了解耦，即他的具体实现我们可以在外部做其他方式的实例，这个时候多态也可以派上用场。可以想成我在类的内部占了个座，至于这个座具体要给谁坐，怎么坐，得等到节目要开始之前再来安排，这样我就可以举办不同的活动了。


    public class Notification {
      private MessageSender messageSender;
      
      public Notification(MessageSender messageSender) {
        this.messageSender = messageSender;
      }
      
      public void sendMessage(String cellphone, String message) {
        this.messageSender.send(cellphone, message);
      }
    }
    
    public interface MessageSender {
      void send(String cellphone, String message);
    }
    
    // 短信发送类
    public class SmsSender implements MessageSender {
      @Override
      public void send(String cellphone, String message) {
        //....
      }
    }
    
    // 站内信发送类
    public class InboxSender implements MessageSender {
      @Override
      public void send(String cellphone, String message) {
        //....
      }
    }
    
    //使用Notification
    MessageSender messageSender = new SmsSender();
    Notification notification = new Notification(messageSender);
    
# 3. 依赖反转原则

依赖反转 -- Dependency Inversion Principle 

> High-level modules shouldn’t depend on low-level modules. Both modules should depend on abstractions. In addition, abstractions shouldn’t depend on details. Details depend on abstractions.

高层模块（high-level modules）不要依赖低层模块（low-level）。高层模块和低层模块应该通过抽象（abstractions）来互相依赖。除此之外，抽象（abstractions）不要依赖具体实现细节（details），具体实现细节（details）依赖抽象（abstractions）。
