---
title: Java map() 与 flatMap()的比较
date: 2020-02-05 18:59:48
categories: BackEnd
tags:
    - Java
    - Map
top:
---

`map()`和`flatMap()`方法都来自于functional languages. 在Java8当中，我们可以在Optional, Stream还有CompletableFuture当中找到他们。

Stream代表一些列的对象，而Optional代表一个存在或者空的值。`map()`和`flatmap()`都是聚合方法，尽管其有着相同的返回类型，但是实际上他们有很多的不同。下面我们通过例子来逐一展现。

# 1. 在Optionals当中

    // map()
    Optional<String> s = Optional.of("Test");
    assertEquals(Optional.of("TEST"), s.map(String::toUpperCase));
    
    // 如果情况更为复杂，变成Optional<Optional<String>>
    assertEquals(Optional.of(Optional.of("STRING")), 
        Optional.of("string").map(s -> Optional.of("STRING")));
        
    // 同样的代码用flatmap来表示
    assertEquals(Optional.of("STRING"), 
    Optional.of("string").flatMap(s -> Optional.of("STRING")));
    
# 2. 在Streams当中

map方法只能做一层的序列化，但是flatmap可以做多层的，来解决Stream<Stream<R>>的这种结构的问题。

    // 对于这种多层架构的, map()方法就显得力有未逮了
    List<List<String>> list = Arrays.asList(
      Arrays.asList("a"),
      Arrays.asList("b"));
    System.out.println(list);
    
    // flatmap可以很好的解决
    System.out.println(list
    .stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()));
    
# Reference

1. https://www.baeldung.com/java-difference-map-and-flatmap
2. https://www.mkyong.com/java8/java-8-flatmap-example/
