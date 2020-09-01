---
title: Java数值计算精度问题
date: 2020-08-31 22:29:02
categories: BackEnd
tags:
    - Java
top:
---


# 1. Double 

```
System.out.println(0.1+0.2);
System.out.println(1.0-0.8);
System.out.println(4.015*100);
System.out.println(123.3/100);

double amount1 = 2.15;
double amount2 = 1.10;
if (amount1 - amount2 == 1.05)
    System.out.println("OK");
    
// Output 
0.30000000000000004
0.19999999999999996
401.49999999999994
1.2329999999999999
```

上述问题出现的原因是因为计算机是以二进制存储数值的，浮点数也是如此。 Java采用的是IEEE754标准来实现浮点数的表达和运算，当将一个10进制的数值转化成浮点数的时候，会出现无限循环的结果。当使用二进制表示是无限循环的时候，转换成10进制就会造成精度的缺失了。

# 2. BigDecimal 

BigDecimal可以用于浮点数精确表达的场景，但是使用BigDecimal的时候，一定要注意使用字符串的构造方法来初始化

```
System.out.println(new BigDecimal("0.1").add(new BigDecimal("0.2")));
System.out.println(new BigDecimal("1.0").subtract(new BigDecimal("0.8")));
System.out.println(new BigDecimal("4.015").multiply(new BigDecimal("100")));
System.out.println(new BigDecimal("123.3").divide(new BigDecimal("100")));


0.3
0.2
401.500
1.233

```

+ BigDecimal 
    + 有scale, Precision的概念
    + scale 表示小数点右边的位数
    + precision 表示精度，即有效数字的长度

+ BigDecimal的equals判等
    + 比较的是value和scale 两个值的！



    System.out.println(new BigDecimal("1.0").equals(new BigDecimal("1")))

    false
    
    
```  
/**
 * Compares this {@code BigDecimal} with the specified
 * {@code Object} for equality.  Unlike {@link
 * #compareTo(BigDecimal) compareTo}, this method considers two
 * {@code BigDecimal} objects equal only if they are equal in
 * value and scale (thus 2.0 is not equal to 2.00 when compared by
 * this method).
 *
 * @param  x {@code Object} to which this {@code BigDecimal} is
 *         to be compared.
 * @return {@code true} if and only if the specified {@code Object} is a
 *         {@code BigDecimal} whose value and scale are equal to this
 *         {@code BigDecimal}'s.
 * @see    #compareTo(java.math.BigDecimal)
 * @see    #hashCode
 */
@Override
public boolean equals(Object x)

```

# 3. 数值溢出问题

所有的基本数值类型都有超出表达范围的可能性，而且是没有任何异常的默默的溢出

+ 可以使用Math类的addExact, substractExact等方法进行数值运算，在溢出的时候主动抛出异常
+ 也可以使用BigInteger，也会主动抛出异常