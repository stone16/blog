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
    
