---
title: 'Java8 Date Time API '
date: 2020-02-19 19:08:47
categories: BackEnd
tags:
    - Java
top:
---
# 1. 为什么需要新的Date API
Java8 的一大更新在于终于将Date Time一致化，这解决了在Java8以前我们观察到的非常多的问题：

譬如：
+ Java Date Time类定义在不同的地方，比如在java.util & java.sql里面都有，而样式和格式转化的类都定义在java.text的包里，比较混乱
+ java.util.Date包括date和time类，而java.sql.Date只包含date
+ 并没有清晰定义的类用于处理time, timestamp, formatting, parsing 
+ 所有的Date类都是可变的，并不是线程安全的
+ Date类不支持全球化，没有时区的支持，在java8之前，为了显示当地时间，就得使用java.util.Calendar 还有 java.util.TimeZone,整个变得比较麻烦

# 2. Java8 Date Time API 详解

## 2.1 Packages

+ java.time Package 
    + 这是新的Date Time API的基础包，一些主要的基本类都在这里面，譬如LocalDate, LocalTime, LocalDateTime, Instant, Period, Duration  
+ java.time.chrono 
    + 定义了抽象的API接口，针对于非ISO标准的calendar 系统，我们可以通过extend AbstractChronology类来创建我们自己的calendar系统
+ java.time.format 
    + 包含用来Formatting还有parsing date time对象的类
+ java.time.temporal 
    + 包含一些时间对象，比如找到月份的第一天， 最后一天之类的
+ java.time.zone
    + 支持不同的时区 

## 2.2 LocalDate 

+ Immutable class 
+ 默认样式为 yyyy-MM-dd
+ 我们可以使用`now()`方法来获得当前的日期
+ 也可以通过提供年月日来创建localDate对象
+ 我们同时也可以传入ZoneId来得到在特定的时区的日期


    package com.journaldev.java8.time;
    
    import java.time.LocalDate;
    import java.time.Month;
    import java.time.ZoneId;
    
    /**
     * LocalDate Examples
     * @author pankaj
     *
     */
    public class LocalDateExample {
    
    	public static void main(String[] args) {
    		
    		//Current Date
    		LocalDate today = LocalDate.now();
    		System.out.println("Current Date="+today);
    		
    		//Creating LocalDate by providing input arguments
    		LocalDate firstDay_2014 = LocalDate.of(2014, Month.JANUARY, 1);
    		System.out.println("Specific Date="+firstDay_2014);
    		
    		
    		//Try creating date by providing invalid inputs
    		//LocalDate feb29_2014 = LocalDate.of(2014, Month.FEBRUARY, 29);
    		//Exception in thread "main" java.time.DateTimeException: 
    		//Invalid date 'February 29' as '2014' is not a leap year
    		
    		//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
    		LocalDate todayKolkata = LocalDate.now(ZoneId.of("Asia/Kolkata"));
    		System.out.println("Current Date in IST="+todayKolkata);
    
    		//java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
    		//LocalDate todayIST = LocalDate.now(ZoneId.of("IST"));
    		
    		//Getting date from the base date i.e 01/01/1970
    		LocalDate dateFromBase = LocalDate.ofEpochDay(365);
    		System.out.println("365th day from base date= "+dateFromBase);
    		
    		LocalDate hundredDay2014 = LocalDate.ofYearDay(2014, 100);
    		System.out.println("100th day of 2014="+hundredDay2014);
    	}
    
    }



    // output 
    Current Date=2014-04-28
    Specific Date=2014-01-01
    Current Date in IST=2014-04-29
    365th day from base date= 1971-01-01
    100th day of 2014=2014-04-10

## 2.3 LocalTime 

+ Immutable Class 
+ 表示一个可读的时间 (vs Instant 基本不可读)
+ 默认样式为 hh:mm:ss:zz
+ 和LocalDate基本一致的用法，可以传入参数生成实例，支持时区


    package com.journaldev.java8.time;
    
    import java.time.LocalTime;
    import java.time.ZoneId;
    
    /**
     * LocalTime Examples
     * @author pankaj
     *
     */
    public class LocalTimeExample {
    
    	public static void main(String[] args) {
    		
    		//Current Time
    		LocalTime time = LocalTime.now();
    		System.out.println("Current Time="+time);
    		
    		//Creating LocalTime by providing input arguments
    		LocalTime specificTime = LocalTime.of(12,20,25,40);
    		System.out.println("Specific Time of Day="+specificTime);
    		
    		//Try creating time by providing invalid inputs
    		//LocalTime invalidTime = LocalTime.of(25,20);
    		//Exception in thread "main" java.time.DateTimeException: 
    		//Invalid value for HourOfDay (valid values 0 - 23): 25
    		
    		//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
    		LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));
    		System.out.println("Current Time in IST="+timeKolkata);
    
    		//java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
    		//LocalTime todayIST = LocalTime.now(ZoneId.of("IST"));
    		
    		//Getting date from the base date i.e 01/01/1970
    		LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
    		System.out.println("10000th second time= "+specificSecondTime);
    
    	}
    }


    // Output 
    Current Time=15:51:45.240
    Specific Time of Day=12:20:25.000000040
    Current Time in IST=04:21:45.276
    10000th second time= 02:46:40

## 2.4 LocalDateTime 

+ Immutable date-time object 
+ represent both date and time 
+ default format at yyyy-MM-dd-HH-mm-ss.zzz 
+ 用工厂方法来拿到LocalDate和LocalTime的input 然后来创建LocalDateTime的实例


    package com.journaldev.java8.time;
    
    import java.time.LocalDate;
    import java.time.LocalDateTime;
    import java.time.LocalTime;
    import java.time.Month;
    import java.time.ZoneId;
    import java.time.ZoneOffset;
    
    public class LocalDateTimeExample {
    
    	public static void main(String[] args) {
    		
    		//Current Date
    		LocalDateTime today = LocalDateTime.now();
    		System.out.println("Current DateTime="+today);
    		
    		//Current Date using LocalDate and LocalTime
    		today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
    		System.out.println("Current DateTime="+today);
    		
    		//Creating LocalDateTime by providing input arguments
    		LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
    		System.out.println("Specific Date="+specificDate);
    		
    		
    		//Try creating date by providing invalid inputs
    		//LocalDateTime feb29_2014 = LocalDateTime.of(2014, Month.FEBRUARY, 28, 25,1,1);
    		//Exception in thread "main" java.time.DateTimeException: 
    		//Invalid value for HourOfDay (valid values 0 - 23): 25
    
    		
    		//Current date in "Asia/Kolkata", you can get it from ZoneId javadoc
    		LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
    		System.out.println("Current Date in IST="+todayKolkata);
    
    		//java.time.zone.ZoneRulesException: Unknown time-zone ID: IST
    		//LocalDateTime todayIST = LocalDateTime.now(ZoneId.of("IST"));
    		
    		//Getting date from the base date i.e 01/01/1970
    		LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
    		System.out.println("10000th second time from 01/01/1970= "+dateFromBase);
    
    	}
    }
    
    // Output
    Current DateTime=2014-04-28T16:00:49.455
    Current DateTime=2014-04-28T16:00:49.493
    Specific Date=2014-01-01T10:10:30
    Current Date in IST=2014-04-29T04:30:49.493
    10000th second time from 01/01/1970= 1970-01-01T02:46:40

## 2.5 Instant 

是为了生成机器阅读的时间格式，它会使用unix的时间戳来存储日期和时间

    package com.journaldev.java8.time;
    
    import java.time.Duration;
    import java.time.Instant;
    
    public class InstantExample {
    
    	public static void main(String[] args) {
    		//Current timestamp
    		Instant timestamp = Instant.now();
    		System.out.println("Current Timestamp = "+timestamp);
    		
    		//Instant from timestamp
    		Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli());
    		System.out.println("Specific Time = "+specificTime);
    		
    		//Duration example
    		Duration thirtyDay = Duration.ofDays(30);
    		System.out.println(thirtyDay);
    	}
    
    }

    // Output 
    Current Timestamp = 2014-04-28T23:20:08.489Z
    Specific Time = 2014-04-28T23:20:08.489Z
    PT720H

## 2.6 常用API


    package com.journaldev.java8.time;
    
    import java.time.LocalDate;
    import java.time.LocalTime;
    import java.time.Period;
    import java.time.temporal.TemporalAdjusters;
    
    public class DateAPIUtilities {
    
    	public static void main(String[] args) {
    		
    		LocalDate today = LocalDate.now();
    		
    		//得到年份，看是否为闰年
    		System.out.println("Year "+today.getYear()+" is Leap Year? "+today.isLeapYear());
    		
    		//比较两个时间的先后
    		System.out.println("Today is before 01/01/2015? "+today.isBefore(LocalDate.of(2015,1,1)));
    		
    		//从LocalDate创建LocalDateTime
    		System.out.println("Current Time="+today.atTime(LocalTime.now()));
    		
    		//加减时间的操作
    		System.out.println("10 days after today will be "+today.plusDays(10));
    		System.out.println("3 weeks after today will be "+today.plusWeeks(3));
    		System.out.println("20 months after today will be "+today.plusMonths(20));
    
    		System.out.println("10 days before today will be "+today.minusDays(10));
    		System.out.println("3 weeks before today will be "+today.minusWeeks(3));
    		System.out.println("20 months before today will be "+today.minusMonths(20));
    		
    		//时间上的加减
    		System.out.println("First date of this month= "+today.with(TemporalAdjusters.firstDayOfMonth()));
    		LocalDate lastDayOfYear = today.with(TemporalAdjusters.lastDayOfYear());
    		System.out.println("Last date of this year= "+lastDayOfYear);
    		
    		Period period = today.until(lastDayOfYear);
    		System.out.println("Period Format= "+period);
    		System.out.println("Months remaining in the year= "+period.getMonths());		
    	}
    }



    Year 2014 is Leap Year? false
    Today is before 01/01/2015? true
    Current Time=2014-04-28T16:23:53.154
    10 days after today will be 2014-05-08
    3 weeks after today will be 2014-05-19
    20 months after today will be 2015-12-28
    10 days before today will be 2014-04-18
    3 weeks before today will be 2014-04-07
    20 months before today will be 2012-08-28
    First date of this month= 2014-04-01
    Last date of this year= 2014-12-31
    Period Format= P8M3D
    Months remaining in the year= 8

## 2.7 时间的Parsing 和 Formatting


    package com.journaldev.java8.time;
    
    import java.time.Instant;
    import java.time.LocalDate;
    import java.time.LocalDateTime;
    import java.time.format.DateTimeFormatter;
    
    public class DateParseFormatExample {
    
    	public static void main(String[] args) {
    		
    		//Format examples
    		LocalDate date = LocalDate.now();
    		//default format
    		System.out.println("Default format of LocalDate="+date);
    		//使用特定的Formatter
    		System.out.println(date.format(DateTimeFormatter.ofPattern("d::MMM::uuuu")));
    		System.out.println(date.format(DateTimeFormatter.BASIC_ISO_DATE));
    		
    		
    		LocalDateTime dateTime = LocalDateTime.now();
    		//default format
    		System.out.println("Default format of LocalDateTime="+dateTime);
    		//specific format
    		System.out.println(dateTime.format(DateTimeFormatter.ofPattern("d::MMM::uuuu HH::mm::ss")));
    		System.out.println(dateTime.format(DateTimeFormatter.BASIC_ISO_DATE));
    		
    		Instant timestamp = Instant.now();
    		//default format
    		System.out.println("Default format of Instant="+timestamp);
    		
    		//Parse examples
    		LocalDateTime dt = LocalDateTime.parse("27::Apr::2014 21::39::48",
    				DateTimeFormatter.ofPattern("d::MMM::uuuu HH::mm::ss"));
    		System.out.println("Default format after parsing = "+dt);
    	}
    
    }


    // Output
    
    Default format of LocalDate=2014-04-28
    28::Apr::2014
    20140428
    Default format of LocalDateTime=2014-04-28T16:25:49.341
    28::Apr::2014 16::25::49
    20140428
    Default format of Instant=2014-04-28T23:25:49.342Z
    Default format after parsing = 2014-04-27T21:39:48


# Reference 

1. https://www.journaldev.com/2800/java-8-date-localdate-localdatetime-instant 
