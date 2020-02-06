---
title: Java Pattern Regex表达式
date: 2020-02-05 19:00:32
categories: BackEnd
tags:
    - Java
    - Regex
top:
---
# 1. 为什么要使用Pattern？

一般来说如果我们要对String做某个范式下的替换时，我们需要使用

    stringEG.replaceAll("(?i)@gmail\\.com$", "");
    
上面这行代码是将stringEG最后的@gmail.com给替换掉，通过这种方式来获得用户名。

这种Replace操作我们会经常使用，但是上述有一个问题，即每次运行都要执行一遍Regex操作，这样很费时间，每次都要进行编译，Pattern可以帮助我们解决这个问题。

通过设置static的变量，我们可以将Compile完的结果存起来，然后在需要的时候直接使用这个结果即可。

使用方法如下所示:

    private static final Pattern USER_NAME_PATTERN = Pattern.compile("(?i)@gmail\\.com$);
    
    final String username =  USER_NAME_PATTERN.matcher(stringEG.replace(""));
    
# 2. 如何使用Pattern？

使用Java Pattern，重点在于对于正则表达式的使用，可以看一下文章 - [正则表达式](https://llchen60.com/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/), 里面有对正则的详细介绍。

Pattern对象是一个已经编译过的正则表达式的表达，Pattern类没有public的构造器，想要创建一个Pattern，我们需要首先调用其静态的compile()方法，通过这个方法会得到一个Pattern对象。

Matcher 对象用来解释正则表达式然后根据表达式来找符合规则的相关表达，同样没有public的构造器，通过调用matcher() 方法来作比较

