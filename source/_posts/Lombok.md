---
title: Lombok
date: 2020-02-08 21:17:23
categories: BackEnd
tags:
    - Java
    - Annotation
    - Lombok
top:
---

Merely want to list all annotations here, and do some analysis. Lombok is a great tool to relieve java programmer from writing duplicate code. 

# 1. @EqualsAndHashCode

Generates hashCode and equals implementations from the fields of your object. 

## 1.1 What's hashCode/ equals use for? 

Equals compare pass-in objects' attributes, to see if they are the same. It compares all the field values to make judgement. 

== compares whether two object references point to the same object

Use hashcode() method to optimize performance when comparing objects. ***Execute hashcode() returns a unique ID for each object in your program, which makes the task of comparing the whole state of the object much easier.*** 

First run hashcode() method to judge if two objects are same, then run equals() method. 

See [Comparing Java objects with `equals()` and `hashcode()`](https://www.javaworld.com/article/3305792/learn-java/java-challengers-4-comparing-java-objects-with-equals-and-hashcode.html)

## 1.2 Implementation 

1. By default, it uses all non-static and non-transient fields
2. Do modification by set `@EqualsAndHashCode.Include` or `@EqualsAndHashCode.Exclude`

### 1.3.1 Apply to a class that extends another 

Normally, auto-generating an equals and hashCode method for such classes is a bad idea, as the superclass also defines fields, which also need equals/hashCode code but this code will not be generated. By setting callSuper to true, you can include the equals and hashCode methods of your superclass in the generated methods. For hashCode, the result of super.hashCode() is included in the hash algorithm, and forequals, the generated method will return false if the super implementation thinks it is not equal to the passed in object.  You can safely call your superclass equals if it, too, has a lombok-generated equals method.

`callSuper`, set it to ture when you don't extend anything is a compile time error. 


     import lombok.EqualsAndHashCode;
    
    @EqualsAndHashCode
    public class EqualsAndHashCodeExample {
      private transient int transientVar = 10;
      private String name;
      private double score;
      @EqualsAndHashCode.Exclude private Shape shape = new Square(5, 10);
      private String[] tags;
      @EqualsAndHashCode.Exclude private int id;
      
      public String getName() {
        return this.name;
      }
      
      @EqualsAndHashCode(callSuper=true)
      public static class Square extends Shape {
        private final int width, height;
        
        public Square(int width, int height) {
          this.width = width;
          this.height = height;
        }
      }
    }
    
# 2. @NonNull 
Use this annotation on the parameter of a mothod or constructor to have lombok generate a null-check statement. 

And a @NonNull on a primitive parameter results in a warning. 


     import lombok.NonNull;
    
    public class NonNullExample extends Something {
      private String name;
      
      public NonNullExample(@NonNull Person person) {
        super("Hello");
        this.name = person.getName();
      }
    }
    
# 3. @Cleanup

It's an antomatic resource management: call your close() methods safely with no hassle!

You can use @Cleanup to ***ensure a given resource is automatically cleaned up before the code execution path exits your current scope***. 

For example, you can use it like： 

    @Cleanup InputStream in = new FileInputStream("some/file");
    
As a result, at the end of the scope you are in, `in.close()` is called. The call is guaranteed to run by way of a try/ finally construct. 

If the type of object you'd like to cleanup does not have a close method, you can specify the name of this method like:

    @Cleanup("dispose") org.eclipse.swt.widgets.CoolBar bar = new CoolBar(parent, 0); 
    
Notice: there should be no variables in the cleanup method. 


     import lombok.Cleanup;
    import java.io.*;
    
    public class CleanupExample {
      public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      }
    }
    
# 4. @Getter/ @Setter 

Annotate any field with Getter and Setter to let lombok generate the default getter/ setter automatically. 

The generated getter/setter method will be public unless you explicitly specify an AccessLevel, as shown in the example below. Legal access levels are PUBLIC, PROTECTED, PACKAGE, and PRIVATE.

Also we can put the annotation on a class, in this way, it's as if you annotate all the non-static fields in the class with annotation. 


     import lombok.AccessLevel;
    import lombok.Getter;
    import lombok.Setter;
    
    public class GetterSetterExample {
      /**
       * Age of the person. Water is wet.
       * 
       * @param age New value for this person's age. Sky is blue.
       * @return The current value of this person's age. Circles are round.
       */
      @Getter @Setter private int age = 10;
      
      /**
       * Name of the person.
       * -- SETTER --
       * Changes the name of this person.
       * 
       * @param name The new value.
       */
      @Setter(AccessLevel.PROTECTED) private String name;
      
      @Override public String toString() {
        return String.format("%s (age: %d)", name, age);
      }
    }
    
# 5. @ToString 

No need to start a debugger to see your fields, lombok can generate a toString for you. 

Any ***class*** can be annotated with @ToString to let lombok generate an implementation of the toString() method. 

We can set: 

+ includeFieldNames 
    + Add some clarity to the output 
+ @ToString.Exclude 
+ @ToString(onlyExplicitlyIncluded = true)
    + specify exactly which fields you wish to be used  
    + then marking each field you want to include with `@ToString.Include`
+ Can also include non static methods that take no argument  -> use `@ToString.Include`

    import lombok.ToString;
    
    @ToString
    public class ToStringExample {
      private static final int STATIC_VAR = 10;
      private String name;
      private Shape shape = new Square(5, 10);
      private String[] tags;
      @ToString.Exclude private int id;
      
      public String getName() {
        return this.name;
      }
      
      @ToString(callSuper=true, includeFieldNames=true)
      public static class Square extends Shape {
        private final int width, height;
        
        public Square(int width, int height) {
          this.width = width;
          this.height = height;
        }
      }
    }
    
# 6. @NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor 

Constructors: 
+ Generates constructors that take no arguments
+ Generates constructors that take one argument per final/ non-null field 
+ Generates constructors that take one argument for every field

## 6.1 @NoArgsConstructor

@NoArgsConstructor will generate a constructor with no parameters. 

If it's not possible(because of final fields), a compiler error will result, unless we use `@NoArgsConstructor(force = true)`. Then all final fields are initialized with 0/ false/ null 

## 6.2 @RequiredArgsConstructor 

@RequiredArgsConstructor  generates a constructor with 1 parameter for each field that requires special handling.  All **non-initialized final fields** get a parameter, as well as any fields that are marked as **@NonNull** that aren't initialized where they are declared. For those fields marked with @NonNull, an explicit null check is also generated. The constructor will throw a NullPointerException if any of the parameters intended for the fields marked with @NonNull contain null. The order of the parameters match the order in which the fields appear in your class.

## 6.3 @AllArgsConstructor

@AllArgsConstructor generates a constructor with 1 parameter for each field in your class. Fields marked with @NonNull result in null checks on those parameters.

# 7. @Data 
All togerther: a shortcut for: 

+ @ToString,
+ @EqualsAndHashCode
+ @Getter on all fields
+ @Setter on all non-final fields
+ @RequiredArgsConstructor 


@Data generates all the boilerplate that is normally associated with simple POJOs (Plain Old Java Objects) and beans: getters for all fields, setters for all non-final fields, and appropriate toString, equals and hashCode implementations that involve the fields of the class, and a constructor that initializes all final fields, as well as all non-final fields with no initializer that have been marked with @NonNull, in order to ensure the field is never null.

All generated getters and setters will be public

All fields marked as transient will not be considered for hashCode and equals. All static fields will be skipped entirely. 


     import lombok.AccessLevel;
    import lombok.Setter;
    import lombok.Data;
    import lombok.ToString;
    
    @Data public class DataExample {
      private final String name;
      @Setter(AccessLevel.PACKAGE) private int age;
      private double score;
      private String[] tags;
      
      @ToString(includeFieldNames=true)
      @Data(staticConstructor="of")
      public static class Exercise<T> {
        private final String name;
        private final T value;
      }
    }
    
# 8. @Value 

@Value is the immutable variant of @Data. All fields are made private and final by default. setters are not generated at all. 

The class itself is also made final by default, becuase immutability is not something that can be forced onto a sunclass.

Actually, @Value equals to `final @ToString @EqualsAndHashCode @AllArgsConstructor @FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE) @Getter`


    import lombok.AccessLevel;
    import lombok.experimental.NonFinal;
    import lombok.experimental.Value;
    import lombok.experimental.Wither;
    import lombok.ToString;
    
    @Value public class ValueExample {
      String name;
      @Wither(AccessLevel.PACKAGE) @NonFinal int age;
      double score;
      protected String[] tags;
      
      @ToString(includeFieldNames=true)
      @Value(staticConstructor="of")
      public static class Exercise<T> {
        String name;
        T value;
      }
    }
# 9. @Builder 

The @Builder annotation produces complex builder APIs for your classes.

@Builder lets you automatically produce the code required to have your class be instantiable with code such as:
Person.builder().name("Adam Savage").city("San Francisco").job("Mythbusters").job("Unchained Reaction").build();

# 10. @Getter(lazy = true)

You can let lombok generate a getter which will calculate a value once, the first time this getter is called, and cache it from then on. This can be useful if calculating the value takes a lot of CPU, or the value takes a lot of memory. To use this feature, create a private final variable, initialize it with the expression that's expensive to run, and annotate your field with @Getter(lazy=true)


     import lombok.Getter;
    
    public class GetterLazyExample {
      @Getter(lazy=true) private final double[] cached = expensive();
      
      private double[] expensive() {
        double[] result = new double[1000000];
        for (int i = 0; i < result.length; i++) {
          result[i] = Math.asin(i);
        }
        return result;
      }
    }