---
title: Spring @Transactional
date: 2020-03-24 19:37:03
categories: BackEnd
tags:
    - Spring
    - Java
top:
---
Spring的Transactional注解用来做事务管理

# 1. 使用方法

首先我们需要在xml当中配置事务信息，定义transactionManager的bean，当然也可以使用注解来实现对于bean的定义，具体如下：

    <tx:annotation-driven />
    <bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
    </bean>
    
    @EnableTransactionManagement

而后，在具体的使用的时候，我们只需要将@Transactional注解添加到合适的方法当中，并且设置合适的属性信息

+ name
    + 指定事务管理器
+ propagation
    + 事务的传播行为，默认为REQUIRED
+ isolation 
    + 事务的隔离度，默认为DEFAULT
+ timeout
    + 事务的超时时间，默认为-1，如果超过该时间限制但事务没有完成，就自动回滚
+ read-only
    + 指定事务是否为只读事务，默认值为false
    + 当要忽略那些不需要事务的方法的时候，可以设置read-only为true
+ rollback-for
    + 指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，可以通过逗号来做分隔
+ no-rollback-for
    + 对于在这里定义的exception，不回滚 

# 2. Spring 注解方式的事务实现机制

在调用@Transactional的目标方法之后，Spring Framework会通过AOP代理，在代码运行时生成一个代理对象，根据注解的属性配置信息，决定该声明@Transactional的目标方法是否由拦截器 - TransactionInterceptor来拦截.

如果确定要被拦截，那么就会在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑，最后根据执行情况是否出现异常，利用抽象事务管理器提交或者回滚事务。

![Spring事务实现机制.jpg](https://i.loli.net/2020/03/25/cM5yEjbPAIogk1X.jpg)

# 3. Isolation Level
Isolation是有不同的配置的，它主要是为了避免事务的一些副作用：
+ 脏读：读到同时进行的事务还没有提交上去的数据
+ 不可重复的读：重复读的时候会读到不同的数据，因为有同时进行的事务对同一条数据进行了更新
+ 幽灵读取：在做query的时候，再度执行拿到不同的行，因为有同时进行的事务在做更新

针对不同级别的事务，Spring有如下的设置

+ DEFAULT
+ READ_UNCOMMITTED
    + 最低的隔离水平 
    + 允许大部分的同时的访问
    + 有上述所有的弊端
+ READ_COMMMITED
    + 阻止脏读 
+ REPEATABLE_READ
    + 阻止脏读
    + 阻止不可重复读取
+ SERAILIZABLE
    + 最高程度的隔离
    + 基本是单序列的执行

# Reference 
1. https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html
2. https://dzone.com/articles/how-does-spring-transactional
3. https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html