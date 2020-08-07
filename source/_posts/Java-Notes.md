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