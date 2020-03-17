---
title: SOLID - 接口隔离原则
date: 2020-03-16 18:26:08
categories: SystemDesign
tags:
    - SOLID
top:
---
接口隔离原则 -- interface segregation principle 

客户端不应该强迫依赖它不需要的接口。

# 1. 将接口视为一组API接口的集合


    public interface UserService {
      boolean register(String cellphone, String password);
      boolean login(String cellphone, String password);
      UserInfo getUserInfoById(long id);
      UserInfo getUserInfoByCellphone(String cellphone);
    }
    
    public class UserServiceImpl implements UserService {
      //...
    }

当我们要实现删除操作的时候，最好不要直接在UserService里面加上这个方法，因为用户服务实质上不应该被默认直接具有删除的权限，相对应的，我们应该去创建一个新的接口，里面实现有删除相关的方法，实现接口的隔离。


    public interface UserService {
      boolean register(String cellphone, String password);
      boolean login(String cellphone, String password);
      UserInfo getUserInfoById(long id);
      UserInfo getUserInfoByCellphone(String cellphone);
    }
    
    public interface RestrictedUserService {
      boolean deleteUserByCellphone(String cellphone);
      boolean deleteUserById(long id);
    }
    
    public class UserServiceImpl implements UserService, RestrictedUserService {
      // ...省略实现代码...
    }


# 2. 将接口理解为单个API接口或者函数

函数的设计需要功能单一，不要将多个不同的功能逻辑放在一个函数当中实现。


    public class Statistics {
      private Long max;
      private Long min;
      private Long average;
      private Long sum;
      private Long percentile99;
      private Long percentile999;
      //...省略constructor/getter/setter等方法...
    }
    
    public Statistics count(Collection<Long> dataSet) {
      Statistics statistics = new Statistics();
      //...省略计算逻辑...
      return statistics;
    }
    

count方法算了太多不同的指标，应该将其分割开的。

    public Long max(Collection<Long> dataSet) { //... }
    public Long min(Collection<Long> dataSet) { //... } 
    public Long average(Colletion<Long> dataSet) { //... }
    // ...省略其他统计函数...
    
# 3. 将接口理解为OOP中的接口的概念

即在设计接口的时候尽量减少大而全的接口的设计，每个接口都只做一件事情，这样子类在implement interface的时候，我们通过接口名也可以很清晰的知道在做什么，会有什么功能，也很大程度上提升了代码的复用性。