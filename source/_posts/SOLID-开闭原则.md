---
title: SOLID - 开闭原则
date: 2020-03-10 20:52:23
categories: SystemDesign
tags: SOLID
top:
---
开闭原则说的是对扩展开放，对修改关闭。我们希望做到的是使得代码有着比较好的扩展性，但是同时也不会影响到他的可读性。

# 1. 如何理解 对扩展开放，对修改关闭？ 

Open closed principle, Software entities (modules, classes, functions, etc) should be open for extension, but closed for modification. 

添加一个新的功能应该是在已有代码基础上扩展代码，而非修改已有的代码。

# 2. 实例解释

代码实现的功能就是当TPS超过某个预设的最大值的时候，或者是错误的数量超过允许的最大值，那么就会触发警报
    public class Alert {
      private AlertRule rule;
      private Notification notification;
    
      public Alert(AlertRule rule, Notification notification) {
        this.rule = rule;
        this.notification = notification;
      }
    
      public void check(String api, long requestCount, long errorCount, long durationOfSeconds) {
        long tps = requestCount / durationOfSeconds;
        if (tps > rule.getMatchedRule(api).getMaxTps()) {
          notification.notify(NotificationEmergencyLevel.URGENCY, "...");
        }
        if (errorCount > rule.getMatchedRule(api).getMaxErrorCount()) {
          notification.notify(NotificationEmergencyLevel.SEVERE, "...");
        }
      }
    }
    
现在我们需要添加一个功能，当每秒钟请求数量超过某个阈值的时候，我们也要触发警告，发送通知。

如果直接在上述代码中进行修改的话，我们主要是需要在check函数当中，添加一个新的输入参数，timeoutCount，表示超时的请求数量，然后再check函数里面加上对应的逻辑


    public class Alert {
      // ...省略AlertRule/Notification属性和构造函数...
      
      // 改动一：添加参数timeoutCount
      public void check(String api, long requestCount, long errorCount, long timeoutCount, long durationOfSeconds) {
        long tps = requestCount / durationOfSeconds;
        if (tps > rule.getMatchedRule(api).getMaxTps()) {
          notification.notify(NotificationEmergencyLevel.URGENCY, "...");
        }
        if (errorCount > rule.getMatchedRule(api).getMaxErrorCount()) {
          notification.notify(NotificationEmergencyLevel.SEVERE, "...");
        }
        // 改动二：添加接口超时处理逻辑
        long timeoutTps = timeoutCount / durationOfSeconds;
        if (timeoutTps > rule.getMatchedRule(api).getMaxTimeoutTps()) {
          notification.notify(NotificationEmergencyLevel.URGENCY, "...");
        }
      }
    }
    
这样的改动是有不少问题的，比如我们对于传入参数做了改动，那所有调用这个函数的地方都需要进行修改，这就是个不小的问题了；而且在修改了check函数以后，所有的单元测试都需要进行修改，会很麻烦的。

为了提高这整个类的拓展性，我们可以做以下的重构操作：
+ 将check函数的多个入参封装成ApiStatInfo类
+ 引入handler的概念，将if判断逻辑分散到各个handler当中



    public class Alert {
      private List<AlertHandler> alertHandlers = new ArrayList<>();
      
      public void addAlertHandler(AlertHandler alertHandler) {
        this.alertHandlers.add(alertHandler);
      }
    
      public void check(ApiStatInfo apiStatInfo) {
        for (AlertHandler handler : alertHandlers) {
          handler.check(apiStatInfo);
        }
      }
    }
    
    public class ApiStatInfo {//省略constructor/getter/setter方法
      private String api;
      private long requestCount;
      private long errorCount;
      private long durationOfSeconds;
    }
    
    public abstract class AlertHandler {
      protected AlertRule rule;
      protected Notification notification;
      public AlertHandler(AlertRule rule, Notification notification) {
        this.rule = rule;
        this.notification = notification;
      }
      public abstract void check(ApiStatInfo apiStatInfo);
    }
    
    public class TpsAlertHandler extends AlertHandler {
      public TpsAlertHandler(AlertRule rule, Notification notification) {
        super(rule, notification);
      }
    
      @Override
      public void check(ApiStatInfo apiStatInfo) {
        long tps = apiStatInfo.getRequestCount()/ apiStatInfo.getDurationOfSeconds();
        if (tps > rule.getMatchedRule(apiStatInfo.getApi()).getMaxTps()) {
          notification.notify(NotificationEmergencyLevel.URGENCY, "...");
        }
      }
    }
    
    public class ErrorAlertHandler extends AlertHandler {
      public ErrorAlertHandler(AlertRule rule, Notification notification){
        super(rule, notification);
      }
    
      @Override
      public void check(ApiStatInfo apiStatInfo) {
        if (apiStatInfo.getErrorCount() > rule.getMatchedRule(apiStatInfo.getApi()).getMaxErrorCount()) {
          notification.notify(NotificationEmergencyLevel.SEVERE, "...");
        }
      }
    }
    
在使用的时候，如下：


    public class ApplicationContext {
      private AlertRule alertRule;
      private Notification notification;
      private Alert alert;
      
      public void initializeBeans() {
        alertRule = new AlertRule(/*.省略参数.*/); //省略一些初始化代码
        notification = new Notification(/*.省略参数.*/); //省略一些初始化代码
        alert = new Alert();
        alert.addAlertHandler(new TpsAlertHandler(alertRule, notification));
        alert.addAlertHandler(new ErrorAlertHandler(alertRule, notification));
      }
      public Alert getAlert() { return alert; }
    
      // 饿汉式单例
      private static final ApplicationContext instance = new ApplicationContext();
      private ApplicationContext() {
        instance.initializeBeans();
      }
      public static ApplicationContext getInstance() {
        return instance;
      }
    }
    
    public class Demo {
      public static void main(String[] args) {
        ApiStatInfo apiStatInfo = new ApiStatInfo();
        // ...省略设置apiStatInfo数据值的代码
        ApplicationContext.getInstance().getAlert().check(apiStatInfo);
      }
    }
    
    
需要修改的地方：


    public class Alert { // 代码未改动... }
    public class ApiStatInfo {//省略constructor/getter/setter方法
      private String api;
      private long requestCount;
      private long errorCount;
      private long durationOfSeconds;
      private long timeoutCount; // 改动一：添加新字段
    }
    public abstract class AlertHandler { //代码未改动... }
    public class TpsAlertHandler extends AlertHandler {//代码未改动...}
    public class ErrorAlertHandler extends AlertHandler {//代码未改动...}
    // 改动二：添加新的handler
    public class TimeoutAlertHandler extends AlertHandler {//省略代码...}
    
    public class ApplicationContext {
      private AlertRule alertRule;
      private Notification notification;
      private Alert alert;
      
      public void initializeBeans() {
        alertRule = new AlertRule(/*.省略参数.*/); //省略一些初始化代码
        notification = new Notification(/*.省略参数.*/); //省略一些初始化代码
        alert = new Alert();
        alert.addAlertHandler(new TpsAlertHandler(alertRule, notification));
        alert.addAlertHandler(new ErrorAlertHandler(alertRule, notification));
        // 改动三：注册handler
        alert.addAlertHandler(new TimeoutAlertHandler(alertRule, notification));
      }
      //...省略其他未改动代码...
    }
    
    public class Demo {
      public static void main(String[] args) {
        ApiStatInfo apiStatInfo = new ApiStatInfo();
        // ...省略apiStatInfo的set字段代码
        apiStatInfo.setTimeoutCount(289); // 改动四：设置tiemoutCount值
        ApplicationContext.getInstance().getAlert().check(apiStatInfo);
    }

# 3. 一些思考

+ 写代码的时候就需要想想有可能会有哪些需求上的变更，如何设计代码的结构，留好扩展点。
+ 在识别出可变部分之后，要将可变部分封装起来，隔离变化，提供抽象化的不可变接口，给上层系统来使用。当具体的实现发生变化的时候，我们只需要基于相同的抽象接口，扩展一个新的实现，这样子上游的代码就几乎不需要修改了

+ 基于接口而非实现的编程， 对扩展开放，对修改关闭

    // 这一部分体现了抽象意识
    public interface MessageQueue { //... }
    public class KafkaMessageQueue implements MessageQueue { //... }
    public class RocketMQMessageQueue implements MessageQueue {//...}
    
    public interface MessageFormatter { //... }
    public class JsonMessageFormatter implements MessageFormatter {//...}
    public class MessageFormatter implements MessageFormatter {//...}
    
    public class Demo {
      private MessageQueue msgQueue; // 基于接口而非实现编程
      public Demo(MessageQueue msgQueue) { // 依赖注入
        this.msgQueue = msgQueue;
      }
      
      // msgFormatter：多态、依赖注入
      public void sendNotification(Notification notification, MessageFormatter msgFormatter) {
        //...    
      }
    }