---
title: Java Annotations 注解详解
date: 2020-02-05 18:57:11
categories: BackEnd
tags:
    - Java
    - Annotations
top:
---


Java annotation很有用，这篇博客会带着大家去理解Annotation的语法以及用法，希望能有所裨益。

# 1. Annotation架构

## 1.0 Annotation的介绍
用一句话来解释Annotation的话，我想可以称其为Metadata(元数据) - 即数据的数据。

举个例子，比如这里我使用了@Override：

    @Override
    public String toString() {
    return "This is String Representation of current object.";
    }

通过注解的方式，我告诉编译器我重写了这个toString()的方法。但其实就算我没有加这个注解，代码仍然是可以有效运行的，那么用注解的好处有哪些呢？为什么我们需要使用注解呢？？？ 

上述的注解告诉编译器我重写了父类里面的一个方法，然后编译器会去父类检查看这个方法是否存在，如果不存在的话，那就会扔出一个编译错误。通过这种方式我们可以减少错误的发生，并且提高整个代码的可读性，给我们带来一些便利。

其实就像上面说的那样，注解是一种携带元数据的方式，我们实际上在注解之前是用XML来做元数据的存储的。二者各有自己的适用场景。对于XML来说，如果你写的应用有大量的常数，参量，用XML会更好，因为我们可以将这些常量和代码完全解耦，这样在改变常量值的时候会方便很多。如果你想对外暴露一些方法来做服务，那注解会是更好的选择。因为这种情形下元数据最好和方法紧密相连，让开发者意识到这个方法的一些特征。

另外注解提供了一个标准的在代码中定义元数据的方式，在注解之前工程师常会自己定义，比如注释，接口，等等。注解将这个过程做了标准化。

## 1.1 Annotation 组成部分
![fig1.jpg](fig1.jpg)
+ Annotation
    + 1 RetentionPolicy 每一个注解对象都会有唯一的RetentionPolicy属性
    + 1 - n ElementType  每个注解对象都可以有若干个ElementType属性
    + Annotation有多个实现类，包括
        + Deprecated 
        + Documented
        + Inherited 
        + Override 
        + Retention 
        + Target 
        
第一个很重要的观念，就是***注解只是元数据，它不包括任何真正的代码逻辑***。

第二个很重要的观念，建立在第一个的基础之上，即如果注解不包括代码逻辑，那么就一定有针对注解的消费者，来读取注解提供的信息，并且执行对应的代码逻辑。

比如以@override为例，这里JVM就是这个注解的消费者，并且在字节码的水平利用注解信息。
### 1.1.1 Annotation.java

    package java.lang.annotation;
    public interface Annotation {
    
        boolean equals(Object obj);
    
        int hashCode();
    
        String toString();
    
        Class<? extends Annotation> annotationType();
    }


一个接口
### 1.1.2 ElementType.java

    package java.lang.annotation;
    
    public enum ElementType {
        TYPE,               /* 类、接口（包括注释类型）或枚举声明  */
    
        FIELD,              /* 字段声明（包括枚举常量）  */
    
        METHOD,             /* 方法声明  */
    
        PARAMETER,          /* 参数声明  */
    
        CONSTRUCTOR,        /* 构造方法声明  */
    
        LOCAL_VARIABLE,     /* 局部变量声明  */
    
        ANNOTATION_TYPE,    /* 注释类型声明  */
    
        PACKAGE             /* 包声明  */
    }

枚举类型，用来指定Annotation的类型。就是这个注解对象是用来修饰什么的，是方法，是变量，还是其他的各种...

### 1.1.3 RetentionPolicy.java 

    package java.lang.annotation;
    public enum RetentionPolicy {
        SOURCE,            /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */
    
        CLASS,             /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */
    
        RUNTIME            /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
    }

RetentionPolicy是Enum枚举类型，每个都有其对应的行为。

**RetentionPolicy.SOURCE** – Discard during the compile. These annotations don’t make any sense after the compile has completed, so they aren’t written to the bytecode. Examples @Override, @SuppressWarnings

**RetentionPolicy.CLASS** – Discard during class load. Useful when doing bytecode-level post-processing. Somewhat surprisingly, this is the default.

**RetentionPolicy.RUNTIME** – Do not discard. The annotation should be available for reflection at runtime. This is what we generally use for our custom annotations.

## 1.2 通用定义

    @Documented
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyAnnotation1 {
    }
    
### 1.2.1 @interface

意味着实现了java.lang.annotation.Annotation接口，即该注解就是一个Annotation。Annotation接口的实现细节都由编译器来完成的，通过@interface定义注解吼，该注解不能继承其他的注解或接口。

### 1.2.2 @Documented

类和方法的Annotation在缺省情况下是不出现在javadoc中的。如果使用@Documented修饰该Annotation，则表示它可以出现在javadoc中。

### 1.2.3 @Target(ElementType.TYPE)

ElementType是Annotation的类型属性，而@Target的作用，就是来指定类型属性的

有@Target，则该Annotation只能用于其所指定的地方；若没有，则该Annotation可以用于任何地方

### 1.2.4 @Retention(RetentionPolicy.RUNTIME)

RetentionPolicy是Annotation的策略属性，


## 1.3 常用Annotation

### 1.3.1 @Deprecated 
    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Deprecated {
    }

1. @interface -- 它的用来修饰Deprecated，意味着Deprecated实现了java.lang.annotation.Annotation接口；即Deprecated就是一个注解。
2. @Documented -- 它的作用是说明该注解能出现在javadoc中。
3. @Retention(RetentionPolicy.RUNTIME) -- 它的作用是指定Deprecated的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将Deprecated的信息保留在.class文件中，并且能被虚拟机读取。
4.  @Deprecated 所标注内容，不再被建议使用。
### 1.3.2 @Override 

### 1.3.3 @Documented 

### 1.3.4 @Inherited 
用来标注Annotation类型，所标注的Annotation具有继承性

    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.ANNOTATION_TYPE)
    public @interface Inherited {
    }

1. @interface -- 它的用来修饰Inherited，意味着Inherited实现了java.lang.annotation.Annotation接口；即Inherited就是一个注解。
2. @Documented -- 它的作用是说明该注解能出现在javadoc中。
3. @Retention(RetentionPolicy.RUNTIME) -- 它的作用是指定Inherited的策略是RetentionPolicy.RUNTIME。这就意味着，编译器会将Inherited的信息保留在.class文件中，并且能被虚拟机读取。
4. @Target(ElementType.ANNOTATION_TYPE) -- 它的作用是指定Inherited的类型是ANNOTATION_TYPE。这就意味着，@Inherited只能被用来标注“Annotation类型”。

### 1.3.5 @Retention
用来标注Annotation类型，用来指定RetentionPolicy属性

### 1.3.6 @Target
@Target只能被用来标注“Annotation类型”，而且它被用来指定Annotation的ElementType属性。

### 1.3.7 @SuppressWarnings 

@SuppressWarnings 所标注内容产生的警告，编译器会对这些警告保持静默。

    @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
    @Retention(RetentionPolicy.SOURCE)
    public @interface SuppressWarnings {
        String[] value();
    }
    
1. @interface -- 它的用来修饰SuppressWarnings，意味着SuppressWarnings实现了java.lang.annotation.Annotation接口；即SuppressWarnings就是一个注解。
2. @Retention(RetentionPolicy.SOURCE) -- 它的作用是指定SuppressWarnings的策略是RetentionPolicy.SOURCE。这就意味着，SuppressWarnings信息仅存在于编译器处理期间，编译器处理完之后SuppressWarnings就没有作用了。
3. @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE}) -- 它的作用是指定SuppressWarnings的类型同时包括TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE。
- TYPE意味着，它能标注“类、接口（包括注释类型）或枚举声明”。
- FIELD意味着，它能标注“字段声明”。
- METHOD意味着，它能标注“方法”。
- PARAMETER意味着，它能标注“参数”。
- CONSTRUCTOR意味着，它能标注“构造方法”。
- LOCAL_VARIABLE意味着，它能标注“局部变量”。
4.  String[] value(); 意味着，SuppressWarnings能指定参数
5.  SuppressWarnings 的作用是，让编译器对“它所标注的内容”的某些警告保持静默。例如，"@SuppressWarnings(value={"deprecation", "unchecked"})" 表示对“它所标注的内容”中的 “SuppressWarnings不再建议使用警告”和“未检查的转换时的警告”保持沉默。
# 2. 创建自己的Annotation

注解只支持基本类型，String，还有Enum。所有的注解的属性都被定义为方法，default的值也会在方法里提供。


    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @interface Todo {
    public enum Priority {LOW, MEDIUM, HIGH}
    public enum Status {STARTED, NOT_STARTED}
    String author() default "Yash";
    Priority priority() default Priority.LOW;
    Status status() default Status.NOT_STARTED;
    }

这里我们创建了一个新的注解，@Todo


    @Todo(priority = Todo.Priority.MEDIUM, author = "Yashwant", status = Todo.Status.STARTED)
    public void incompleteMethod1() {
        //一些业务逻辑
    }

值得注意的一点，如果注解只有一个属性，那么它应该被命名为value,在使用它的时候不用使用具体的变量名了。

    @interface Author{
    String value();
    }
    @Author("Yashwant")
    public void someMethod() {
    }
    
定义好注解以后，我们需要写注解的消费者，使用反射。

    Class businessLogicClass = BusinessLogic.class;
    for(Method method : businessLogicClass.getMethods()) {
        Todo todoAnnotation = (Todo)method.getAnnotation(Todo.class);
        if(todoAnnotation != null) {
        System.out.println(" Method Name : " + method.getName());
        System.out.println(" Author : " + todoAnnotation.author());
        System.out.println(" Priority : " + todoAnnotation.priority());
        System.out.println(" Status : " + todoAnnotation.status());
        }
    }
# 3. Reference

1. https://www.cnblogs.com/skywang12345/p/3344137.html 
2. https://dzone.com/articles/how-annotations-work-java