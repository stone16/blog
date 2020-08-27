---
title: Java 判等问题
date: 2020-08-26 13:16:43
categories: BackEnd
tags:
    - Java
top:
---
# 1. equals vs `==` 

+ 对于基本类型，应使用==，比较的是值 
+ 对于引用类型，需要使用equals，进行内容判等。使用`==`判断的是指针 --> 代表的是两个对象在内存中的地址

这里要注意的是Java是有字符串常量池机制的，当代码中出现双引号形式创建字符串对象的时候，JVM会先对字符串进行检查，如果字符串常量池存在相同内容的字符串对象的引用，就将这个引用返回；否则就创建新的字符串对象，然后将这个引用放入字符串常量池当中，并返回该引用

另外一个小坑是Integer在[-128,127]之间的数值是会做缓存的，即对于这中间的数值，即便你直接用`==`进行判断，有可能是直接会过的...


    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
## 1.1 equals方法的实现

+ equals在Object类当中的定义比较的是对象的引用


    public boolean equals(Object obj) {
        return (this == obj);
    }
    
+ 而Integer，String类都重写了这个方法


```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

上述代码是先比较了引用，如果引用的地址一致，那么久可以直接返回true了。如果不一致，那就首先判断类的类型，如果是String类，再进行长度判断，如果长度一致，就逐个比较字符

+ 实现一个equals方法，需要注意
    + 首先进行指针判断，如果对象相同直接返回true
    + 需要对另一方进行判空，空对象和自身的比较结果一定是false
    + 需要判断两个对象的类型，如果类型都不同，那么直接返回false
    + 在确保类型相同的情况下进行类型的强制转换，然后逐一判断所有字段
        + 需要进行类型强制转换是因为我们override的equals方法默认的输入参数是Object


## 1.2 使用Lombok的小坑

Lombok的@Data注解会帮助我们实现equals和hashcode方法，但是有继承关系的时候，Lombok自动生成的方法是不会考虑到父类的

+ 对于不想进行equals和hashCode判断的参数，可以使用：
    + `@EqualsAndHashCode.Exclude`

+ 对于想要使用父类属性的场景，可以使用
    + `@EqualsAndHashCode(callSuper = true)`