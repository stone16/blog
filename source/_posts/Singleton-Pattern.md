---
title: Singleton Pattern
date: 2022-12-07 19:18:26
categories: Design Pattern
tags:
top:
---
# 1. why we need

- Registry Setting
- Thread Pool / Connection Pool

> The singleton Pattern ensures a class has only one instance, and provides a global point of access to it.
> 
- let a class to manage a single instance of itself
    - also prevent other classes from creating a new instance on its own
- provide a global access point to the instance

# 2. Implementation

## 2.1 1st Iteration

```java
public class Singleton {
	private static Singleton uniqueInstance;

	private Singleton() {}

	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton();
		}
		return uniqueInstance;
	}
}

```

- Issue when multi threading

Thread A 

if (uniqueInstance == null) { // return true 

new Singleton() 

Thread B 

if (uniqueInstance == null) { // return true 

new Singleton() 

- Under such case, we’ll create two new instances

## 2.2 Implementation with multi-threading support

```java
public class Singleton {
	private static Singleton uniqueInstance;

	private Singleton() {}

	public static synchronized Singleton getInstance() {
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton();
		}
		return uniqueInstance;
	}
}

```

- synchronized
    - force every thread to wait its turn before it can enter the method
    - no two threads may enter the method at the same time
- Issue
    - we only need synchronized when there is no instance, once there are, no need to use synchronized, it’s just overhead

## 2.3  Implementation with multi-threading support  — Eagerly created instance

```java
public class Singleton {
	private static Singleton uniqueInstance = new Singleton();

	private Singleton() {}

	public static synchronized Singleton getInstance() {
		return uniqueInstance;
	}
}
```

- We rely on the JVM to create the unique instance of the Singleton when the class is loaded
- JVM guarantees the instance will be created before any thread accesses the static unique instance variable

## 2.4  Implementation with multi-threading support  — Double Checked Locking

```java
public class Singleton {
	private volatile static Singleton uniqueInstance;

	private Singleton() {}

	public static synchronized Singleton getInstance() {
		if (uniqueInstance == null) {
			synchronized (Singleton.class) {
				if (uniqueInstance == null) {
					uniqueInstance = new Singleton();
				}
			}
		}
		return uniqueInstance;	
	}
}
```

- volatile
    - ensure multiple threads handle the unique instance variable correctly when it is initialized to the singleton instance
    

## 2.5 Implementation with multi-threading support — Using Enum

```java
public enum Singleton {
	UNIQUE_INSTANCE;
}

public class SingletonClient {
	Singleton singleton = Singleton.UNIQUE_INSTANCE;
}
```

ClassLoader 

- Responsible for loading java classes dynamically to the JVM during runtime
- Responsible for loading classes into memory

type 

- application class loader
    - **An application or system class loader loads our own files in the classpath.**
- extension class loader
    - **Extension class loaders load classes that are an extension of the standard core Java classes.**
- bootstrap class loader
    - **A bootstrap or primordial class loader is the parent of all the others.**
    - **This is because the bootstrap class loader is written in native code, not Java, so it doesn't show up as a Java class**
    
- how does it work
    - JVM request a class
    - class loader try to locate the class and load the definition into the runtime using **fully qualified class name**
- delegation model
    - delegate the search of the class/ resource to the parent class loader
    

# Reference

1. [https://www.baeldung.com/java-classloaders](https://www.baeldung.com/java-classloaders) 
2. [https://learning.oreilly.com/library/view/head-first-design/9781492077992/ch05.html](https://learning.oreilly.com/library/view/head-first-design/9781492077992/ch05.html)