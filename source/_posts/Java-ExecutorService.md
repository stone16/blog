---
title: Java - ExecutorService
date: 2020-02-05 18:58:50
categories: BackEnd
tags:
    - Java
    - Executor Service
top:
---
ExecutorService is a framework provided by the JDK which simplifies the execution of tasks in ***asynchronous*** mode. ExecutorService automatically provides a pool of threads and API for assigning tasks to it. 
# 1. Instantiation 
## 1.1 Factory methods of Executors class
Use its factory methods of the Executors class to create ExecutorService. 

    ExecutorService executor = Executors.newFixedThreadPool(10);

## 1.2 Directly create

    ExecutorService executorService = 
      new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,   
      new LinkedBlockingQueue<Runnable>());
# 2. Assigning Tasks 

ExecutorService can execute Runnable and Callable tasks. 

    Runnable runnableTask = () -> {
        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    };
     
    Callable<String> callableTask = () -> {
        TimeUnit.MILLISECONDS.sleep(300);
        return "Task's execution";
    };
     
    List<Callable<String>> callableTasks = new ArrayList<>();
    callableTasks.add(callableTask);
    callableTasks.add(callableTask);
    callableTasks.add(callableTask);

## 2.1 execute()

The execute() method is void, and it doesn’t give any possibility to get the result of task’s execution or to check the task’s status (is it running or executed).

    executerService.execute(runnableTask);
    
## 2.2 submit()

submit() submits a Callable or a Runnable task to an ExecutorService and returns a result of type Future.

    Future<String> future = executorService.submit(callableTask);
    
## 2.3 invokeAny()

invokeAny() assigns a collection of tasks to an ExecutorService, causing each to be executed, and returns the result of a successful execution of one task (if there was a successful execution).

    String result = executorService.invokeAny(callableTasks);
    
## 2.4 invokeAll()

invokeAll() assigns a collection of tasks to an ExecutorService, causing each to be executed, and returns the result of all task executions in the form of a list of objects of type Future.

    List<Future<String>> futures = executorService.invokeAll(callableTasks);
    
# 3. Shutdown 

In general, the ExecutorService will not be automatically destroyed when there is not task to process. It will stay alive and wait for new work to do.

In some cases this is very helpful; for example, if an app needs to process tasks which appear on an irregular basis or the quantity of these tasks is not known at compile time.

On the other hand, an app could reach its end, but it will not be stopped because a waiting ExecutorService will cause the JVM to keep running.

## 3.1 shutdown()

The shutdown() method doesn’t cause an immediate destruction of the ExecutorService. It will make the ExecutorService stop accepting new tasks and shut down after all running threads finish their current work.

    executorService.shutdown();
    
## 3.2 shutdownNow()

The shutdownNow() method tries to destroy the ExecutorService immediately, but it doesn’t guarantee that all the running threads will be stopped at the same time. This method returns a list of tasks which are waiting to be processed. It is up to the developer to decide what to do with these tasks.

    List<Runnable> notExecutedTasks = executorService.shutDownNow();
    
## 3.3 best behavior

    executorService.shutdown();
    try {
        if (!executorService.awaitTermination(800, TimeUnit.MILLISECONDS)) {
            executorService.shutdownNow();
        } 
    } catch (InterruptedException e) {
        executorService.shutdownNow();
    }
    
# 4. Future interface 

Future interface provides a `get()` which returns an actual result of the Callable task's execution or null in the case of Runnable task. Calling the get() method while the task is still running will cause execution to block until the task is properly executed and the result is available.

    Future<String> future = executorService.submit(callableTask);
    String result = null;
    try {
        result = future.get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
    
With very long blocking caused by the get() method, an application’s performance can degrade. If the resulting data is not crucial, it is possible to avoid such a problem by using timeouts:

    String result = future.get(200, TimeUnit.MILLISECONDS);
    

some other methods provided:

    cancel()
    isCancelled()
    isDone() 

# 5. How to sync the value across different threads? 

Suppose we have i++ in several threads, and they all perform such operations. To make the i computed properly, we need to make it atomic. 

Use `AtomicInteger` or `synchronized` to get the final correct result. 

Notice, `volatile` cannot make sure the final result is correct. It mainly makes sure the visibility of newest value, but there is possibility that we mound and switch to other thread before the new value being recorded. 

AtomicInteger class uses CAS(Compare and swap) low level CPU operations. They allow you to modify a particular variable only if the present value is equal to something else (and is returned successfully).

# 6. Executor.execute() and ExecutorService.submit() differences 

1. execute(Runnable) does not return anything; while submit(Callable<T>) returns a Future object which allows a way to programatically cancel the running thread and get the return result. 
2. submit() can accept both Runnable and Callable task but execute() can only accept the Runnable task
3. submit() return a Future object while execute() has no return
4. get() is a blocking call, which will take some time