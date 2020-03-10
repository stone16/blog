---
title: SOLID - 单一职责原则
date: 2020-03-09 20:49:07
categories: SystemDesign
tags: SOLID 
top:
---

# 1. Intro

+ Single Responsibility Principle - 单一职责原则
    + 我们希望对于一个类或者模块来说，他们都是只有一个职责的，即只完成一个功能
    + 这个职责说的其实就是，不要设计大而全的类，应当设计粒度小，功能单一的类。
    + 在同一个类当中的方法功能，应该是在同一个业务方向当中的

# 2. 如何判断类的职责是否单一

+ E.G
    + 如果一个类，既要处理和其他微服务的交互，又要到数据库query，那么这就是两个职责，我们就应该将这个类分割开，划分到两个不同的类当中去 

+ E.G2


    public class UserInfo {
      private long userId;
      private String username;
      private String email;
      private String telephone;
      private long createTime;
      private long lastLoginTime;
      private String avatarUrl;
      private String provinceOfAddress; // 省
      private String cityOfAddress; // 市
      private String regionOfAddress; // 区 
      private String detailedAddress; // 详细地址
      // ...省略其他属性和方法...
    }

对于上述UserInfo类来说，里面全是User的一些属性，但是其中有将近半数是关于地址的，我们有理由将其细分为UserAddress类以及UserInfo类，但是在实际应用场景当中，我们需要考虑我们到底是准备如何使用这些数据的。

如果应用场景就是拿出用户相关的信息，那放在一起无伤大雅，但是如果是要做物流，电商的场景，那么我们单独想拿出地址相关信息的场景就会比较多了，这种情况最好就分成两个不同的类了。

实际开发当中，实际上一般情况下都是先写一个粗粒度的类，以满足业务的需求，随着业务的发展，如果粗粒度的类越来越庞大，我们就可以将这个粗粒度的类拆分成几个更细粒度的类，即--持续重构。

# 3. 判断是否满足单一职责原则的判断准则

+ 类中代码的行数，函数或者属性过多，会影响代码的可读性和可维护性，需要考虑对类进行拆分了
+ 类依赖的其他类过多，或者依赖类的其他类过多，不符合高内聚，低耦合的设计思想，我们需要对其考虑进行拆分
+ 私有方法过多，我们需要考虑是否应该将私有方法独立到新的类当中，设置为public方法，供更多的类使用，从而提高代码的复用性
+ 如果对于一个类，比较难取名字，只能用相对泛泛的名字，那很可能意味着这个类的职责有点过多了
+ 类中的大量方法都是集中操作类中的某几个属性，那么就可以考虑将这几个属性和对应的方法拆分出来


需要注意的一点是： 我们使用这些准则，亦或者是设计模式，最终的目的还是提高代码的可读性，可扩展性，复用性，可维护性等。这才应当是我们判断是否要采用SOLID准则，以及其他的原则的最终标准。