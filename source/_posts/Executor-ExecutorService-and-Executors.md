---
title: 'Executor, ExecutorService and Executors'
date: 2020-02-04 09:11:48
categories: BackEnd
tags:
    - Java 
top:
---
Executor, ExecutorService and Executors, they are all part of Java's Executor framework, this framework offers a threadpool. Thus we don't need to manage threads on our own, the pool can help us manage themselves. 

A thread pool which is created when an application is a startup solves both of these problems. It has **ready threads** to serve clients when needed and it also has a bound on how many threads to create under load.


# 1. Executor

**the core interface** which is an abstraction for parallel execution

It separates task from execution, this is different from java.lang.Thread class which **combines both task and its execution**. 


# 2. ExecutorService

ExecutorService is an extension of Executor interface and provides a facility for returning a Future object and terminate, or shut down the thread pool. Once the shutdown is called, the thread pool will not accept new task but complete any pending task. It also provides a submit() method which extends Executor.execute() method and returns a Future.

The Future object provides the facility of asynchronous execution, which means you don't need to wait until the execution finishes, you can just submit the task and go around, come back and check if Future object has the result, if execution is completed then it would have result which you can access by using the Future.get() method. Just remember that this method is a **blocking method** i.e. it will wait until execution finish and the result is available if it's not finished already.

By using the Future object returned by ExecutorService.submit() method, you can also cancel the execution if you are not interested anymore. It provides cancel() method to cancel any pending execution.


# 3. Executors 

Third one Executors is a utility class similar to Collections, which provides **factory methods** to create different types of thread pools e.g. fixed and cached thread pools. Let's see some more difference between these three classes.

# 4. Difference 

1) One of the key difference between Executor and ExecutorService interface is that **former is a parent interface while ExecutorService extends Executor** i.e. it's a sub-interface of Executor.

2) Another important difference between ExecutorService and Executor is that Executor defines execute() method which accepts an object of the Runnable interface, while submit() method can accept objects of both Runnable and Callable interfaces.


3) The third difference between Executor and ExecutorService interface is that execute() method doesn't return any result, its return type is void but submit() method returns the result of computation via a Future object. This is also the key difference between submit() and execute() method, which is one of the frequently asked Java concurrency interview questions.


4) The fourth difference between ExecutorService and Executor interface is that apart from allowing a client to submit a task, **ExecutorService also provides methods to control the thread pool** e.g. terminate the thread pool by calling the shutDown() method. You should also read "Java Concurrency in Practice" to learn more about the graceful shutdown of a thread-pool and how to handle pending tasks.

5) Executors class provides factory methods to create different kinds of thread pools e.g. newSingleThreadExecutor() creates a thread pool of just one thread, newFixedThreadPool(int numOfThreads) creates a thread pool of fixed number of threads and newCachedThreadPool() creates new threads when needed but reuse the existing threads if they are available.

# 5. Differences between Executor and Thread

1. Executor provides a thread pool in java, while Thread not. 
2. java.lang.Thread is a class in Java while java.util.concurrent.Executor is an interface.
3. The Executor concept is actually an abstraction over parallel computation. It allows concurrent code to be run in managed way. On the other hand, Thread is a concrete way to run the code in parallel.
4. Executor decouples a task (the code which needs to be executed in parallel) from execution, while in the case of a Thread, both task and execution are tightly coupled.
5. The Executor concept allows your task is to be executed by a worker thread from the thread pool, while Thread itself execute your task
6. a Thread can only execute one Runnable task but an Executor can execute any number of Runnable task.