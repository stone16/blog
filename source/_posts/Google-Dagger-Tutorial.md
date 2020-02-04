---
title: Google Dagger Tutorial
date: 2020-02-04 09:12:52
categories: BackEnd
tags:
    - Java
top:
---


# 1. Comparisons
Spring is a giant collection of libraries and utilities. with a lot of integration, an XML configuration, runtime/ reflective bindings. Application already using Spring can use its dependency injection smoothly. 

Dependency is only a small part of it. Guice and Dagger is lightweight and only a dependency injection framework

Dagger is very lightweight framework with very few integrations, java interface/ annotation configuration, and compile-time code generated bindings. 

For dependency injection and IOC container, there is [a post in Chinese](https://llchen60.com/IOC%E5%AE%B9%E5%99%A8%E5%92%8CDependency-Injection%E6%A8%A1%E5%BC%8F/) wrote last year.


Also a big difference between those DI framework is when does the injection happen, compile time or runtime? 

Run-time DI is **based on reflection** which is simpler to use but slower at run-time ---- Spring, Guice 

Compile-time DI is **based on code generation**. This means that all the heavy-weight operations are performed during compilation. It adds complexity but generally performs faster

# 2. Implementation 

+ POJO
+ Module
    + a class provides or builds the objects' dependencies 
+ Component
    + an interface used to generate the injector 


## 2.1 POJO 

    public class Car {
     
        private Engine engine;
        private Brand brand;
     
        @Inject
        public Car(Engine engine, Brand brand) {
            this.engine = engine;
            this.brand = brand;
        }
     
        // getters and setters
    }

## 2.2 Module 

    @Module // similar to @Controller  @Service 
    public class VehiclesModule {
        @Provides // similar to @Bean 
        public Engine provideEngine() {
            return new Engine();
        }
     
        @Provides
        @Singleton
        public Brand provideBrand() { 
            return new Brand("lol"); 
        }
    }

## 2.3 Component 
here we could return the real object we want to be the starting point of the whole mechanism:

Dagger will start from here, go through all the @Inject and satisfy those dependencies. In our example, will create engine and brand object. 

    @Singleton
    @Component(modules = VehiclesModule.class)
    public interface VehiclesComponent {
        Car buildCar();
    }

## 2.4 client side 
Notice: `DaggerVehiclesComponent` is created by dagger automatically. 


    VehiclesComponent component = DaggerVehiclesComponent.create();
    Car eg = component.buildCar();

# 3. Dagger User Guide 

## 3.1 Declaring Dependencies 

Dagger constructs instances of application classes and satisfies their dependencies. It uses @Inject annotation to identify which constructors and fields it is interested in. 

## 3.2 Satisfying Dependencies 

By default, Dagger satisfies each dependency by constructing an instance of the requested type. It call `new SomeObject()` and setting its injectable fields. 

+ @Inject 
    + interfaces cannot be constructed 
    + third party classes cannot be annotated 
    + configurable objects must be configured 
+ Instead, use @provides 
    + all @provides methods should be named with a provide prefix and module classes are named with a Module suffix 

## 3.3 Building the Graph

The @Inject and @Provides annotated classes form a graph of objects, linked by their dependencies. Build the application by **an interface with methods that have no arguments and return the desired type**. 

# Reference 
1. https://stackoverflow.com/questions/39688830/why-use-develop-guice-when-you-have-spring-and-dagger
2. https://www.baeldung.com/dagger-2
3. https://rskupnik.github.io/dependency-injection-in-pet-project-dagger2
4. https://dagger.dev/