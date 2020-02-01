---
title: Tomcat - 连接器
date: 2020-02-01 14:24:18
categories: Web
tags:
    - Tomcat
top:
---
Unix系统下的I/O模型共有5种: 
+ 同步阻塞I/O 
+ 同步非阻塞I/O
+ I/O多路复用
+ 信号驱动I/O
+ 异步I/O

关于连接器，我们将主要从Tomcat如何实现几种I/O手段来入手进行详解。

# 1. Java I/O 模型

I/O是指计算机内存与外部设备之间拷贝数据的过程。CPU是先把外部设备的数据读到内存里，然后再进行处理的。

Java I/O模型之间的不同之处在于在数据从外部设备拷贝到内存的过程当中，CPU闲置，是继续给当前进程使用，还是把CPU给其他进程使用呢。

在Java I/O模型当中，网络通信过程会涉及到两个对象：
+ 调用该I/O操作的用户线程
    + 用户线程等待内核将数据从网卡拷贝到内核空间
    + 内核将数据从内核空间拷贝到用户空间
+ 操作系统内核

各个I/O模型实现这两个步骤的方式是不一样的。

## 1.1 同步阻塞I/O 

用户线程发起read调用吼就阻塞了，让出CPU。内核等待网卡数据到来，把数据从网卡拷贝到内核空间，接着把数据拷贝到用户空间，再把用户线程叫醒。

![fig1.jpg](https://i.loli.net/2020/02/02/nPOsYLSymTGZKBf.jpg)

## 1.2 同步非阻塞I/O

用户线程不断发起read调用，数据没到内核空间时，每次都返回失败，直到数据到了内核空间，这一次read调用后，在等待数据从内核空间拷贝到用户空间这段时间，线程还是阻塞的，等数据到了用户空间再把线程叫醒。 

![fig2.jpg](https://i.loli.net/2020/02/02/8eqioQx14zkZWjK.jpg)

## 1.3 I/O多路复用

用户线程的读取操作分为两步，线程首先发起select调用，目的是问内核数据准备好了没有，等内核把数据准备好，用户线程再发起read调用。在等待数据从内核空间拷贝到用户空间的这段时间里，线程还是阻塞的。

多路复用的意思是一次select调用会向内核查多个数据通道的状态，因此叫做多路复用。

![fig3.jpg](https://i.loli.net/2020/02/02/I3GH1PBRrwnfE9Q.jpg)

## 1.4 异步I/O

用户线程发起read调用的同时注册一个回调函数，read立即返回，等内核将数据准备好以后，再调用指定的回调函数完成处理。这个过程中，用户线程一直没有阻塞。

# 2. NioEndpoint组件

Tomcat的NioEndpoint组件实现了I/O多路复用模型。

## 2.1 总体工作流程

+ 创建一个selector，注册各种事件，然后调用select方法，等待感兴趣的事情发生
+ 感兴趣的事情发生了，就创建一个新的线程从Channel中读取数据。


![fig4.jpg](https://i.loli.net/2020/02/02/8caQkFDGKJinjgy.jpg)

NioEndpoint共有5个组件，分别是：
+ LimitLatch
    + 连接控制器
        + 负责控制最大连接数
        + 一般设定为10000
+ Acceptor
    + 跑在一个独立的线程当中
    + 在一个死循环里调用accept方法来接收新连接，一旦有新的连接请求到来，accept方法返回一个Channel对象，接着把Channel对象交给Poller去处理
    
+ Poller
    + Poller本质上是一个selector, 也跑在单独线程里。Poller在内部维护一个Channel数组，在一个死循环里不断检测Channel的数据就绪状态，一旦有Channel可读，就生成一个SocketProcessor任务对象扔给Executor去处理 
+ SocketProcessor
    + run方法会调用Http11Processor来读取和解析请求数据 
+ Executor 
    + 线程池
    + 负责运行SocketProcessor任务类

## 2.2 LimitLatch 

用来控制连接个数，当连接数到达最大时阻塞线程，直到后续组件处理完一个连接后才将连接数减1.到达最大连接数以后操作系统底层还是会接收客户端连接，但用户层已经不再接收了。

    public class LimitLatch {
    // AbstractQueuedSynchronizer在内部维护一个状态和一个线程队列
    // 用来控制线程什么时候挂起，什么时候唤醒
        private class Sync extends AbstractQueuedSynchronizer {
         
            @Override
            protected int tryAcquireShared() {
                long newCount = count.incrementAndGet();
                if (newCount > limit) {
                    count.decrementAndGet();
                    return -1;
                } else {
                    return 1;
                }
            }
    
            // 定义合适唤醒被阻塞的用户线程
            @Override
            protected boolean tryReleaseShared(int arg) {
                count.decrementAndGet();
                return true;
            }
        }
    
        private final Sync sync;
        private final AtomicLong count;
        private volatile long limit;
        
        // 线程调用这个方法来获得接收新连接的许可，线程可能被阻塞 
        // AQS知道是否需要阻塞的逻辑在tryAcquireShared方法当中定义了
        public void countUpOrAwait() throws InterruptedException {
          sync.acquireSharedInterruptibly(1);
        }
    
        // 调用这个方法来释放一个连接许可，那么前面阻塞的线程可能被唤醒
        // 
        public long countDown() {
          sync.releaseShared(0);
          long result = getCount();
          return result;
       }
    }

## 2.3 Acceptor

Acceptor实现了Runnable接口，一个端口号只能对应一个ServerSocketChannel，故而这个Channel是在多个Acceptor线程之间共享的。

    serverSock = ServerSocketChannel.open();
    serverSock.socket().bind(addr,getAcceptCount());
    serverSock.configureBlocking(true);


+ bind 方法的第二个参数表示操作系统的等待队列长度，我在上面提到，当应用层面的连接数到达最大值时，操作系统可以继续接收连接，那么操作系统能继续接收的最大连接数就是这个队列长度，可以通过 acceptCount 参数配置，默认是 100。
+ ServerSocketChannel 被设置成阻塞模式，也就是说它是以阻塞的方式接收连接的。


ServerSocketChannel通过accept()接受新的连接，accept()方法返回获得SocketChannel对象，然后将SocketChannel对象封装在一个PollerEvent对象当中，并将PollerEvent对象压入Poller的Queue当中。

## 2.4 Poller

本质是一个Selector，内部维护一个Queue 

    private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();

使用synchronized关键字来保证在同一时刻只有一个Acceptor线程对Queue进行读写。

Poller不断通过内部的selector对象向内核查询Channel的状态，一旦可读就生成任务类SocketProcessor交给Executor去处理。Poller的另一个重要任务是循环遍历检查自己所管理的SocketChannel是否已经超时，若超时就关闭这个Channel。

## 2.5 SocketProcessor

SocketProcessor任务类会被交给线程池去处理，processor内主要是调用Http11Processor组件来处理请求，http11Processor读取Channel的数据来生成ServletRequest对象

## 2.6 NioEndpoint的高并发思路
对于三大方面的事情各自有一个线程组，可以配置线程数量。
+ 接受连接      Acceptor 
+ 检测I/O事件   Poller
+ 处理请求      Executor 