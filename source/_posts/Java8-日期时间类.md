---
title: Java8 日期时间类
date: 2020-09-28 14:20:22
categories: BackEnd
tags:
top:
---
在Java8之前，处理日期时间需求的时候需要使用Data, Calendar, SimpleDateFormat来声明时间戳，使用日历处理日期和格式化解析日期时间，Java8之后有了新的日期时间类，定义的比原来要清晰很多，且支持线程安全。

这篇文章看看时间错乱问题背后的原因，看看使用遗留的日期时间类，来处理日期时间初始化，格式化，解析，计算等可能会遇到的问题，以及如何使用新日期时间类来解决。

# 初始化日期时间

## 使用Date初始化日期时间

```
Date date = new Date(2019, 12, 31, 11, 12, 13);
System.out.println(date);

// Output 
Sat Jan 31 11:12:13 CST 3920
```

这是因为Date初始化时间的时候是用的和1970的差值，月的值是从0- 11 的

我们可以使用Calendar来定义时区的信息 
# 时区问题

```
Calendar calendar = Calendar.getInstance();
calendar.set(2019, 11, 31, 11, 12, 13);
System.out.println(calendar.getTime());
Calendar calendar2 = Calendar.getInstance(TimeZone.getTimeZone("America/New_York"));
calendar2.set(2019, Calendar.DECEMBER, 31, 11, 12, 13);
System.out.println(calendar2.getTime());
```

+ Date类
    + Date本身没有时区的问题，都是保存的UTC时间
    + Date当中保存的是一个时间戳，是从1970-1-1 0点到现在的毫秒数
    + 保存方法
        + 以UTC保存
        + 以字面量保存
            + 年月日 时分秒  时区信息


+ ZoneId 
    + `ZoneId.of`用来初始化一个标准的时区
    + `ZoneOffset.ofHours` 通过offset来初始化具有指定的时间差的时区

+ LocalDateTime
    + 不带有时区属性，是本地时区的日期时间

+ ZonedDateTime 
    + = LocalDateTime + ZoneId 

+ DateTimeFormatter
    + 可以通过withZone方法直接设置需要使用的时区

+ 因此对于国际化时间的处理，应当使用Java8的日期时间类，通过ZonedDateTime来保存时间

# 日期时间格式化和解析

+ SimpleDateFormat的问题
    + 注意使用SimpleDateFormat的时候，大写的YYYY表示的是week year，即所在的周属于哪一年，yyyy表示的是年
    + static 的SimpleDateFormat可能会出现线程安全的问题，其解析和格式化操作是非线程安全的
    + SimpleDateFormat对于格式不匹配表现的非常宽容，可能会隐藏一些错误，要注意格式上的不同
