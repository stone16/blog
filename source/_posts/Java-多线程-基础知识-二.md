---
title: Java 多线程 基础知识(二)
date: 2020-05-03 10:51:40
categories: BackEnd
tags:
    - Java
    - Multi-threading
top:
---
# 1. 并发容器
## 1.1 ConcurrentHashMap 

+ HashMap不是线程安全的
+ 并发情况下一个可行的方式是使用Collections.synchronizedMap()来包装HashMap。
    + 但问题在于一个全局的锁同步不同线程之间的并发访问，会带来不可忽视的性能问题

+ 故而使用ConcurrentHashMap
    + 读写都能保证较高的性能
    + 读操作时几乎不需要加锁
    + 写操作的时候通过锁分段技术只对所操作的段加锁而不影响客户端对其他段的访问


+ ConcurrentHashMap和HashTable的区别主要体现在实现线程安全的方式上不同
    + 底层数据结构
        + ConcurrentHashMap使用分段的数组和链表
        + Hashtable用数组和链表，数组为主体，链表是为了解决哈希冲突的
    + 线程安全的实现方式
        + 使用node数组 + 链表 + 红黑树的数据结构来实现，并发控制使用synchronized和CAS操作
        + Hashtable是使用synchronized来保证线程安全的，效率相对较低
## 1.2 CopyOnWriteArrayList

+ 针对现实应用场景当中，读操作远远多于写操作，因为读操作不会修改原有数据，所以就不对读进行加锁操作了。允许多个线程同时访问list的内部数据。
+ ReentranReadWriteLock 读写锁是读读共享、写写互斥、读写互斥、写读互斥
+ 而CopyOnWriteArrayList 是读取完全不加锁，写入也不会阻塞读取操作，只有写入和写入之间需要进行同步等待。


+ 如何实现的
    + 所有可变操作（add，set 等等）都是通过创建底层数组的新副本来实现的。当 List 需要被修改的时候，我并不修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了。
    + 从计算机系统的角度来说，实际上是拷贝内存，在新内存完成写操作，并将原先的内存指针指向新的内存，原有的内存就可以被回收掉了


        /** The array, accessed only via getArray/setArray. */
        private transient volatile Object[] array;
        public E get(int index) {
            return get(getArray(), index);
        }
        @SuppressWarnings("unchecked")
        private E get(Object[] a, int index) {
            return (E) a[index];
        }
        final Object[] getArray() {
            return array;
        }

            /**
         * Appends the specified element to the end of this list.
         *
         * @param e element to be appended to this list
         * @return {@code true} (as specified by {@link Collection#add})
         */
        public boolean add(E e) {
            final ReentrantLock lock = this.lock;
            lock.lock();//加锁
            try {
                Object[] elements = getArray();
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len + 1);//拷贝新数组
                newElements[len] = e;
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();//释放锁
            }
        }
        
  
## 1.3 ConcurrentLinkedQueue

Java 提供的线程安全的 Queue 可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是 BlockingQueue，非阻塞队列的典型例子是 ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。 阻塞队列可以通过加锁来实现，非阻塞队列可以通过 CAS 操作实现。

从名字可以看出，ConcurrentLinkedQueue这个队列使用链表作为其数据结构．ConcurrentLinkedQueue 应该算是在高并发环境中性能最好的队列了。它之所有能有很好的性能，是因为其内部复杂的实现。

其中主要使用CAS非阻塞算法来实现


# 2. 乐观锁悲观锁

乐观锁适用于写比较少的情况，即冲突本身发生的可能性就比较低，这样就能省去锁的开销，加大整个系统的吞吐量；但是多写的情况下，会比较容易产生冲突，这样就会导致上层不断进行retry，反倒会降低性能，所以一般多写的场景下用悲观锁比较合适。

## 2.1 乐观锁

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号机制**和**CAS算法**实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

关于CAS算法，实质上就先拿到指定内存上的数据，（读取操作），线程操作处理数据，在要写入之前，再次查询该内存位置上的数据，如果数据一致，那就可以写入，如果数据不一致，就throw exception，告知系统出现了问题。

### 2.1.1 乐观锁实现方式
乐观锁可以使用版本号机制或者CAS算法来进行实现

+ 版本号机制
    + 在数据表中加上数据版本号version字段，表示数据被修改的次数
    + 被修改，version值会+1
    + 当线程A要更新数据时，读数据的同时也会读取version值，提交更新的时候，若刚才读取到的version值和当前数据库的version值相等才更新，否在重试

+ CAS算法
    + compare and swap算法，无锁编程
    + 不使用锁的情况下实现多线程之间的变量同步，即在没有线程被阻塞的情况下实现变量的同步 -- 非阻塞同步 Non=blocking synchronization 
    

### 2.1.2 缺点

+ ABA 问题
    + 一个变量初始值为A，在准备赋值的时候仍为A，但是在这段时间当中它有可能已经被改为了其他的值了，CAS操作会认为它从来没有被修改过
    + 可以使用AtomicStampedReference类，compareAndSet方法首先检查当前引用是否等于预期引用，以及当前标志是否等于预期标志。如果全部相等，就以原子方式将该引用和该标志的值设置为给定的更新值。

+ 循环时间开销大
    + 自旋CAS如果长时间不成功，会给CPU带来很大的执行开销

+ 只能保证一个共享变量的原子操作
    + CAS只对单个共享变量有效，当操作涉及多个共享变量的时候CAS无效
    + AtomicReference这一类能够保证引用对象之间的原子性，可以将多个变量放在一个对象里进行CAS操作

## 2.2 悲观锁

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。
