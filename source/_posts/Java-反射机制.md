---
title: Java 反射机制
date: 2020-02-05 19:02:35
categories: BackEnd
tags:
    - Reflection
    - Java
top:
---


# 1. 介绍
Java反射机制是在运行时用来判定或者修改方法，类，接口的行为的API。

+ 反射可以告诉我们类和对象之间的信息，以及我们可以用类的哪些方法来使用这个对象

+ 通过反射，我们就可以在运行的时候赋予类一个新的对象

# 2. 使用范例
反射可以用来获取关于类，构造器和方法的信息：

+ class: getClass()方法可以用来获取对象属于的类的名字
+ Constructors: getConstructor()可以用来获取对象属于的类的public的构造器
+ getMethods() 可以用来获取对象属于的类的public的方法

    // A simple Java program to demonstrate the use of reflection 
    import java.lang.reflect.Method; 
    import java.lang.reflect.Field; 
    import java.lang.reflect.Constructor; 
    
    // class whose object is to be created 
    class Test 
    { 
        // creating a private field 
        private String s; 
    
        // creating a public constructor 
        public Test() { s = "GeeksforGeeks"; } 
    
        // Creating a public method with no arguments 
        public void method1() { 
            System.out.println("The string is " + s); 
        } 
    
        // Creating a public method with int as argument 
        public void method2(int n) { 
            System.out.println("The number is " + n); 
        } 
    
        // creating a private method 
        private void method3() { 
            System.out.println("Private method invoked"); 
        } 
    } 

    class Demo 
    { 
        public static void main(String args[]) throws Exception 
        { 
            // Creating object whose property is to be checked 
            Test obj = new Test(); 
    
                // 构建类的对象
            // getclass method 
            Class cls = obj.getClass(); 
            System.out.println("The name of class is " + 
                                cls.getName()); 
    
            // Getting the constructor of the class through the 
            // object of the class 
            Constructor constructor = cls.getConstructor(); 
            System.out.println("The name of constructor is " + 
                                constructor.getName()); 
    
            System.out.println("The public methods of class are : "); 
    
            // Getting methods of the class through the object 
            // of the class by using getMethods 
            Method[] methods = cls.getMethods(); 
    
            // Printing method names 
            for (Method method:methods) 
                System.out.println(method.getName()); 
    
            // creates object of desired method by providing the 
            // method name and parameter class as arguments to 
            // the getDeclaredMethod  
            // 这里是通过方法的名字和输入变量来获取对应的方法
            Method methodcall1 = cls.getDeclaredMethod("method2", 
                                                    int.class); 
    
            // invokes the method at runtime 
            // 这里是在运行时执行这个方法 
            methodcall1.invoke(obj, 19); 
    
            // creates object of the desired field by providing 
            // the name of field as argument to the 
            // getDeclaredField method 
            // 运行时获取对应的private的变量
            Field field = cls.getDeclaredField("s"); 
    
            // allows the object to access the field irrespective 
            // of the access specifier used with the field 
            // 将这个变量设成可以获取的
            field.setAccessible(true); 
    
            // takes object and the new value to be assigned 
            // to the field as arguments 
            field.set(obj, "JAVA"); 
    
            // Creates object of desired method by providing the 
            // method name as argument to the getDeclaredMethod 
            Method methodcall2 = cls.getDeclaredMethod("method1"); 
    
            // invokes the method at runtime 
            methodcall2.invoke(obj); 
    
            // Creates object of the desired method by providing 
            // the name of method as argument to the 
            // getDeclaredMethod method 
            Method methodcall3 = cls.getDeclaredMethod("method3"); 
    
            // allows the object to access the method irrespective 
            // of the access specifier used with the method 
            methodcall3.setAccessible(true); 
    
            // invokes the method at runtime 
            methodcall3.invoke(obj); 
        } 
    } 

# 3. 详细分析

反射一言以蔽之，即在运行时拿到class，并创建类对应的对象的方式。这种好处是更具灵活性，劣势是会慢很多，代码会相对难理解些。

从代码本身的角度来讲，是指一部分代码有能力去观察/检查另一部分代码。用已知的部分合理推断出未知的部分，这未知的部分其实是指还不知道的信息。

一般来说在Java里我们都是和注解一起来使用反射的，


# 4. 优劣势

+ 好处
    + 反射是什么呢？当我们的程序在运行时，需要动态的加载一些类这些类可能之前用不到所以不用加载到jvm，而是在运行时根据需要才加载，这样的好处对于服务器来说不言而喻，举个例子我们的项目底层有时是用mysql，有时用oracle，**需要动态地根据实际情况加载驱动类，这个时候反射就有用了**，假设 `com.java.dbtest.myqlConnection`，`com.java.dbtest.oracleConnection`这两个类我们要用，这时候我们的程序就写得比较动态化，通过`Class tc = Class.forName("com.java.dbtest.TestConnection");`通过类的全类名让jvm在服务器中找到并加载这个类，而如果是oracle则传入的参数就变成另一个了。这时候就可以看到反射的好处了，这个动态性就体现出java的特性了！
    + 更具拓展性，可以在运行时获取信息
    + 获取一些private的域的值方便debug
+ 劣势
    + 更慢，有延时
    + 会暴露一些接口
    + 反射会要求运行的许可，当在secure manager下来运行可能不被允许


# Reference

1. https://stackoverflow.com/questions/37628/what-is-reflection-and-why-is-it-useful
2. https://docs.oracle.com/javase/tutorial/reflect/index.html
3. https://docs.oracle.com/javase/tutorial/reflect/class/index.html