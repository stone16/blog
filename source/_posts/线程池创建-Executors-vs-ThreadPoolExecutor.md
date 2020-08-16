---
title: '线程池创建: Executors  vs ThreadPoolExecutor'
date: 2020-08-16 15:17:49
categories: BackEnd
tags: 
    - Executors
    - ThreadPoolExecutor
top:
---
工程上对于线程池的使用必不可少，很多人会选择使用Executors class定义的`newCachedThreadPool`以及`newFixedThreadPool`。这篇博文就稍微分析一下二者适用的场景，以及我们应该使用Executors的方法还是直接调用ThreadPoolExecutor来创建线程池。

首先让我们一起看看二者的源码

```
   /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    
 
 
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    } 
 
```


二者对比，你会发现实际上他们都是调用的ThreadPoolExecutor,只是参数是不一样的。那让我们看看ThreadPoolExecutor的源码

```
    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory and rejected execution handler.
     * It may be more convenient to use one of the {@link Executors} factory
     * methods instead of this general purpose constructor.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

```

看一下其中需要的几个参数：

+ corePoolSize
    + 在线程池中最少要保持的线程数量，哪怕已经超过了定义的keepAliveTime 

+ maximumPoolSize
    + 线程池允许的最大线程数量

+ keepAliveTime
    + 当当前线程数超过核心线程数量的时候，就会检查闲置的线程，如果在这段时间没有新的任务，就暂停当前线程

+ unit
    + 定义事件单位

+ workQueue
    + 在任务还没有执行之前，被用来持有这些任务的
    + queue之后持有execute方法提交的Runnable任务


带着这些信息我们再来看Executors.newFixedThreadPool的定义，方法传入了线程数量，然后核心线程数和最大线程数被设为一样的数值，让我们来看看在不同情况下他的表现：

+ 任务数小于等于设定的线程数
    + 一切运行正常
    + 限制的线程不会被关闭

+ 任务数大于设定的线程数
    + 任务会加入到队列当中，进行等待
    + 值得注意的是在实例化LinkedBlockingQueue的时候，传入的参数是`this(Integer.MAX_VALUE);`
        + 这意味着如果任务在线程中执行的时间非常长，任务可以在队列中堆积到无限大，最终结果会是内存被占满..程序崩溃


而对于Executors.newCachedThreadPool来说，其定义的核心线程数量为0，最大线程数是`Integer.MAX_VALUE`,即理论上是可以有无限多的线程，keepAliveTime是60秒，使用的是SynchrounousQueue。

+ 当任务进来的时候
    + 会增加线程
    + 有多少任务进来，就会使用ThreadFactory开多少线程，因为允许的最大线程数时无限大，所以可以一直这么开下去
    + 而其workqueue是SynchrounousQueue,其大小始终为0，在这里我们可以直接任务当任务进来的时候，如果没有空闲的线程，会直接让ThreadFactory来构建新的线程了
    + 那么当任务无限多的时候，就会创建无数多的线程，直接撑爆内存了


由此可以看出来使用Executors的两个方法直接构建线程池因为设定的参数是无界的，可能会导致OOM的错误，更好的方式是自己根据当前线程池的应用场景，来设定参数。

根据应用场景的不同，根据doc，我们有三大类的queue可以选择，分别为：

+ `Synchronous queue`
    + 直接讲任务交给线程
    + 自己本身不持有任何任务的
    + 针对的应用场景可以是各个线程之间任务的执行有某些内在的联系，阻碍一个的执行可能会影响另外一个
    + 为了不拒绝新的线程的创建，就必须设定线程池的大小为Integer.MAX_VALUE
    + 这样如果处理速度低于新任务的提交速度的话，可能会导致非常非常大的线程池

+ `LinkedBlockingQueue`
    + 使用没有边界的queue
    + 这样当所有核心线程都忙碌的时候，任务就都会在队列当中排队
    + 这种方式可以环节突发性的峰值，但是如果处理速度慢于任务堆积的速度，queue会变得很大

+ `ArrayBlockingQueue`
    + 有限长的queue
    + 这样可以防止资源耗尽，但是也很难做调整和优化
    + 队列的大小和最大线程数相互影响，很难做到优化
    + 使用大队列，小线程池可以减少对于CPU的使用，线程切换的损耗，但是单位时间处理速度不会太高
    + 使用小队列，大线程池可以让CPU更忙碌，但是切换线程会有不小的损耗
        
# Reference
1. https://www.ibm.com/developerworks/library/j-jtp0730/index.html 
2. https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html