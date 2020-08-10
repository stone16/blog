---
title: Java Notes
date: 2020-08-03 19:19:37
categories: BackEnd
tags:
    - Java
top:
---
# 1. 并发

## 1.1 ThreadLocal复用问题

ThreadLocal适用于变量在线程间隔离，而在方法或类之间共享的场景。如果用户信息的获取比较昂贵，那么在ThreadLocal中缓存数据时比较合适的做法。


    private static final ThreadLocal<Integer> currentUser = ThreadLocal.withInitial(() -> null);


    @GetMapping("wrong")
    public Map wrong(@RequestParam("userId") Integer userId) {
        //设置用户信息之前先查询一次ThreadLocal中的用户信息
        String before  = Thread.currentThread().getName() + ":" + currentUser.get();
        //设置用户信息到ThreadLocal
        currentUser.set(userId);
        //设置用户信息之后再查询一次ThreadLocal中的用户信息
        String after  = Thread.currentThread().getName() + ":" + currentUser.get();
        //汇总输出两次查询结果
        Map result = new HashMap();
        result.put("before", before);
        result.put("after", after);
        return result;
    }

上述例子当中，我们在设置前设置后都做了记录，来看threadLocal当中都记录了什么信息，。值得注意的是程序是运行在Tomcat当中的，执行程序的线程是Tomcat的工作线程，而Tomcat的工作线程是基于线程池的。

即会重用几个固定的线程，一旦线程重用，那么很可能首次从ThreadLocal获取的值是之前其他用户的请求遗留的值。这时ThreadLocal中的用户信息就是其他用户的信息了。

Take Away: 
1. 代码中没用多线程不以为着你的程序没有使用多线程，Tomcat的Web服务器的业务代码，本身就运行在一个多线程环境当中
2. 使用线程池处理数据就意味着线程是会被重用的，使用类似ThreadLocal工具来存放一些数据的时候，需要注意在代码运行完之后，显式去清空设置的数据。

修正复用的问题的bug： 

    private static final ThreadLocal<Integer> currentUser = ThreadLocal.withInitial(() -> null);

    @GetMapping("right")
    public Map right(@RequestParam("userId") Integer userId) {
        String before  = Thread.currentThread().getName() + ":" + currentUser.get();
        currentUser.set(userId);
        try {
            String after = Thread.currentThread().getName() + ":" + currentUser.get();
            Map result = new HashMap();
            result.put("before", before);
            result.put("after", after);
            return result;
        } finally {
            //在finally代码块中删除ThreadLocal中的数据，确保数据不串
            currentUser.remove();
        }
    }
    
## 1.2 ConcurrentHashMap 

+ ConcurrentHashMap是线程安全的哈希表容器，这里的线程安全是指原子性读写操作是线程安全的。
+ 例子 -- 10个线程一起来补充总共100个元素进去

```
    //线程个数
    private static int THREAD_COUNT = 10;
    //总元素数量
    private static int ITEM_COUNT = 1000;

    //帮助方法，用来获得一个指定元素数量模拟数据的ConcurrentHashMap
    private ConcurrentHashMap<String, Long> getData(int count) {
        return LongStream.rangeClosed(1, count)
                .boxed()
                .collect(Collectors.toConcurrentMap(i -> UUID.randomUUID().toString(), Function.identity(),
                        (o1, o2) -> o1, ConcurrentHashMap::new));
    }

    @GetMapping("wrong")
    public String wrong() throws InterruptedException {
        ConcurrentHashMap<String, Long> concurrentHashMap = getData(ITEM_COUNT - 100);
        //初始900个元素
        log.info("init size:{}", concurrentHashMap.size());

        ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
        //使用线程池并发处理逻辑
        forkJoinPool.execute(() -> IntStream.rangeClosed(1, 10).parallel().forEach(i -> {
            //查询还需要补充多少个元素
            int gap = ITEM_COUNT - concurrentHashMap.size();
            log.info("gap size:{}", gap);
            //补充元素
            concurrentHashMap.putAll(getData(gap));
        }));
        //等待所有任务完成
        forkJoinPool.shutdown();
        forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
        //最后元素个数会是1000吗？
        log.info("finish size:{}", concurrentHashMap.size());
        return "OK";
    }
```

这样子执行的结果就是加入远远超过预期的数量，因为ConcurrentHashMap可以保证多个worker工作的时候不会互相干扰，但是无法保证看到的当前ConcurrentHashMap数据数量的同步

+ Take Aways
    + 使用ConcurrentHashMap，不代表对其多个操作之间的状态是一致的，是没有其他线程在操作它的，如果需要确保，需要手动加锁
    + 诸如size,isEmpty和containsValue等聚合方法，在并发情况下可能会反映ConcurrentHashMap的**中间状态**，因此在并发情况下，***这些方法的返回值只能用作参考，而不能用于流程控制**

解决方案就是通过加锁，使得同时只有一个线程可以操作ConcurrentHashMap

```
@GetMapping("right")
public String right() throws InterruptedException {
    ConcurrentHashMap<String, Long> concurrentHashMap = getData(ITEM_COUNT - 100);
    log.info("init size:{}", concurrentHashMap.size());


    ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
    forkJoinPool.execute(() -> IntStream.rangeClosed(1, 10).parallel().forEach(i -> {
        //下面的这段复合逻辑需要锁一下这个ConcurrentHashMap
        synchronized (concurrentHashMap) {
            int gap = ITEM_COUNT - concurrentHashMap.size();
            log.info("gap size:{}", gap);
            concurrentHashMap.putAll(getData(gap));
        }
    }));
    forkJoinPool.shutdown();
    forkJoinPool.awaitTermination(1, TimeUnit.HOURS);


    log.info("finish size:{}", concurrentHashMap.size());
    return "OK";
}
```

+ 充分使用ConcurrentHashMap的特性
    + 例如面对一个使用Map来统计Key出现次数的场景
    + key范围为10， 最多使用10个并发，循环操作1000万次，每次操作累加随机的key
    + 如果key不存在的话，首次设置值为1 


```
    //循环次数
    private static int LOOP_COUNT = 10000000;
    //线程数量
    private static int THREAD_COUNT = 10;
    //元素数量
    private static int ITEM_COUNT = 10;
    private Map<String, Long> normaluse() throws InterruptedException {
        ConcurrentHashMap<String, Long> freqs = new ConcurrentHashMap<>(ITEM_COUNT);
        ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
        forkJoinPool.execute(() -> IntStream.rangeClosed(1, LOOP_COUNT).parallel().forEach(i -> {
            //获得一个随机的Key
            String key = "item" + ThreadLocalRandom.current().nextInt(ITEM_COUNT);
                    synchronized (freqs) {      
                        if (freqs.containsKey(key)) {
                            //Key存在则+1
                            freqs.put(key, freqs.get(key) + 1);
                        } else {
                            //Key不存在则初始化为1
                            freqs.put(key, 1L);
                        }
                    }
                }
        ));
        forkJoinPool.shutdown();
        forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
        return freqs;
    }
```

但是实际上ConcurrentHashMap本身是使用的Java自带的CAS操作的，在虚拟机层面确保了写入数据的原子性，比加锁的效率高很多，因此相较于直接加synchronized重量锁，我们可以通过computeIfAbsent()操作，和线程安全累加器LongAdder来更有效率的实现我们的统计目的

```
private Map<String, Long> gooduse() throws InterruptedException {
    ConcurrentHashMap<String, LongAdder> freqs = new ConcurrentHashMap<>(ITEM_COUNT);
    ForkJoinPool forkJoinPool = new ForkJoinPool(THREAD_COUNT);
    forkJoinPool.execute(() -> IntStream.rangeClosed(1, LOOP_COUNT).parallel().forEach(i -> {
        String key = "item" + ThreadLocalRandom.current().nextInt(ITEM_COUNT);
                //利用computeIfAbsent()方法来实例化LongAdder，然后利用LongAdder来进行线程安全计数
                freqs.computeIfAbsent(key, k -> new LongAdder()).increment();
            }
    ));
    forkJoinPool.shutdown();
    forkJoinPool.awaitTermination(1, TimeUnit.HOURS);
    //因为我们的Value是LongAdder而不是Long，所以需要做一次转换才能返回
    return freqs.entrySet().stream()
            .collect(Collectors.toMap(
                    e -> e.getKey(),
                    e -> e.getValue().longValue())
            );
}
```

+ 上述代码中，直接使用了ConcurrentHashMap的原子性方法computeIfAbsent来做符合逻辑操作，判断Key是否存在Value，如果不存在则把Lambda表达式运行后的结果放入Map作为Value
+ LongAdder是线程安全的累加器，因此可以直接调用其increment()方法来做累加。

## 1.3 锁

+ 加锁前需要知道锁和被保护的对象是不是一个层面上的
    + 静态字段属于类，需要类级别的锁来进行保护
    + 非静态字段属于类实例，实例级别的锁就可以保护

```
// 定义一个静态int字段counter和一个非静态的wrong方法，实现counter字段的累加操作
class Data {
    @Getter
    private static int counter = 0;
    
    public static int reset() {
        counter = 0;
        return counter;
    }

    public synchronized void wrong() {
        counter++;
    }
}


// 测试代码
@GetMapping("wrong")
public int wrong(@RequestParam(value = "count", defaultValue = "1000000") int count) {
    Data.reset();
    //多线程循环一定次数调用Data类不同实例的wrong方法
    IntStream.rangeClosed(1, count).parallel().forEach(i -> new Data().wrong());
    return Data.getCounter();
}

```

输出结果，因为默认运行100万次，但是页面输出的并不会是100万。

+ 在非静态的wrong方法上加锁，只能够保证多个线程无法执行同一个实例的wrong方法，但无法保证其不会执行不同实例的wrong方法。而静态的counter是被共享的
+ 解决方案时保证在一个实例的方法操作静态变量的时候，其他的实例无法操作这个静态变量

```
class Data {
    @Getter
    private static int counter = 0;
    private static Object locker = new Object();

    public void right() {
        synchronized (locker) {
            counter++;
        }
    }
}
```


+ 除此以外，对锁可以做的优化还包括
    + 精细化锁应用的范围
    + 区分读写场景以及资源的访问冲突，考虑使用悲观锁还是乐观锁
        + 对于读写比例差异明显的场景，考虑使用ReentrantReadWriteLock细化区分读写锁，来提高性能
        + 如果共享资源冲突概率不大，可以考虑使用StampedLock的乐观读的特性，进一步提高性能

## 1.4 线程池

开发当中，我们会使用各种池化技术来缓存创建昂贵的对象，比如线程池，连接池，内存池。一般是预先创建一些对象放入到池当中，使用的时候直接取出使用，用完归还以便复用。通过一定的策略调整池中缓存对象的数量，实现池的动态伸缩。

+ 应当手动进行线程池的声明
    + Java Executors定义了一些快捷的工具办法，来帮助我们快速创建线程池
    + 应当禁止使用这些方法来创建线程池，应当手动new ThreadPoolExecutor来创建线程池
        + 资源耗尽导致OOM问题
            + newFixedThreadPool
            + newCachedThreadPool


### 1.4.1 newFixedThreadPool OOM 问题

```
    @GetMapping("oom1")
    public void oom1() throws InterruptedException {

        ThreadPoolExecutor threadPool = (ThreadPoolExecutor) Executors.newFixedThreadPool(1);
        //打印线程池的信息，稍后我会解释这段代码
        printStats(threadPool); 
        for (int i = 0; i < 100000000; i++) {
            threadPool.execute(() -> {
                String payload = IntStream.rangeClosed(1, 1000000)
                        .mapToObj(__ -> "a")
                        .collect(Collectors.joining("")) + UUID.randomUUID().toString();
                try {
                    TimeUnit.HOURS.sleep(1);
                } catch (InterruptedException e) {
                }
                log.info(payload);
            });
        }

        threadPool.shutdown();
        threadPool.awaitTermination(1, TimeUnit.HOURS);
    }
```

+ 日志显示出现了OOM
+ newFixedThreadPool源码：

```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

    public class LinkedBlockingQueue<E> extends AbstractQueue<E>
            implements BlockingQueue<E>, java.io.Serializable {
        ...


        /**
         * Creates a {@code LinkedBlockingQueue} with a capacity of
         * {@link Integer#MAX_VALUE}.
         */
        public LinkedBlockingQueue() {
            this(Integer.MAX_VALUE);
        }
    ...
    }
```

+ 直接使用了一个LinkedBlockingQueue，而默认构造方法是一个Integer.MAX_VALUE长度的队列，是无界的。
+ 尽管使用newFixedThreadPool可以把工作线程控制在固定的数量上，但任务队列是无界的。如果任务比较多并且执行比较慢的话，队列可能会迅速积压，撑爆内存导致OOM

### 1.4.2 newCachedThreadPool OOM问题

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
                                  
```

+ 线程池最大线程数为Integer.MAX_VALUE，是没有上限的，其工作队列SynchronizedQueue是一个没有存储空间的阻塞队列。
+ SynchronousQueue是没有存储空间的阻塞队列，有请求到来的时候，必须要找到一条工作线程来处理，如果当前没有空闲的线程就再创建一条新的


### 1.4.3 线程池配置Best Practice

+ 根据自己的场景，并发情况来评估线程池的几个核心参数，需要设置有界的工作队列和可控的线程数
    + 核心线程数
    + 最大线程数
    + 线程回收策略
    + 工作队列的类型
    + 拒绝策略

+ 为线程池指定有意义的名称，来方便问题的排查，当出现线程数暴增，线程死锁，线程占用大量CPU这类问题的时候，会抓取线程栈来进行分析，这个时候有意义的线程名称，可以很大程度上方便我们对问题的定位
+ Metrics， alarm来观察线程池的状态



+ 线程池特性
    + 不会初始化corePoolSize个线程，有任务来了才创建工作线程
    + 当核心线程满了之后不会立即扩容线程池，而是把任务堆积到工作队列当中
    + 当工作队列满了之后扩容线程池，一直到线程个数达到maximumPoolSize为止
    + 如果队列已满其达到了最大线程后还有任务来，就按照拒绝策略来处理
    + 当线程数大于核心线程数时，线程等待KeepAliveTime后还没有任务需要处理的话，收缩线程到核心线程数


```
@GetMapping("right")
public int right() throws InterruptedException {
    //使用一个计数器跟踪完成的任务数
    AtomicInteger atomicInteger = new AtomicInteger();
    //创建一个具有2个核心线程、5个最大线程，使用容量为10的ArrayBlockingQueue阻塞队列作为工作队列的线程池，使用默认的AbortPolicy拒绝策略
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(
            2, 5,
            5, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(10),
            new ThreadFactoryBuilder().setNameFormat("demo-threadpool-%d").get(),
            new ThreadPoolExecutor.AbortPolicy());

    printStats(threadPool);
    //每隔1秒提交一次，一共提交20次任务
    IntStream.rangeClosed(1, 20).forEach(i -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        int id = atomicInteger.incrementAndGet();
        try {
            threadPool.submit(() -> {
                log.info("{} started", id);
                //每个任务耗时10秒
                try {
                    TimeUnit.SECONDS.sleep(10);
                } catch (InterruptedException e) {
                }
                log.info("{} finished", id);
            });
        } catch (Exception ex) {
            //提交出现异常的话，打印出错信息并为计数器减一
            log.error("error submitting task {}", id, ex);
            atomicInteger.decrementAndGet();
        }
    });

    TimeUnit.SECONDS.sleep(60);
    return atomicInteger.intValue();
}
```