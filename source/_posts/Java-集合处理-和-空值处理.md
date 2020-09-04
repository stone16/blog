---
title: Java 集合处理 和 空值处理
date: 2020-09-03 20:48:11
categories: BackEnd
tags:
    - Java
top:
---
# 1. `Arrays.asList`

业务开发当中，我们常常会将原始的数组转换为List类数据结构，来继续展开各种Stream操作

+ Arrays.asList无法转换基本类型的数组，可以使用Arrays.stream来进行转换

+ Arrays.asList返回的list是不支持增删操作的，其返回的List是Arrays的内部类ArrayList。内部继承自AbstractList，没有覆写父类的add方法

+ 对原始数组的修改会影响到我们获得的那个List
    + ArrayList实际上是使用了原始的数组，因此在使用的时候，最好再使用New ArrayList来实现解耦


# 2. 空值处理

## 2.1 NullPointerException

+ 可能出现的场景
    + 参数值是Integer等包装类型，使用时因为自动拆箱出现了空指针异常
    + 字符串比较
    + ConcurrentHashMap这种容器不支持Key和Value为null，强行put null的key或Value会出现空指针异常
    + 方法或远程服务返回的list是null，没做判空就直接调用，出现空指针异常
    + 联级调用的null check


+ best practice
    + `string.equalsTo(variableName)`
    + `Optional.ofNullable()`
    + `orElse()`
