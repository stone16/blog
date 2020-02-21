---
title: '工程师需要知道的latency 数字 '
date: 2020-02-20 21:08:38
categories: SystemDesign
tags:
    - latency
    - system design
top:
---
看到一篇博客，叙述了当前内存对于数据的处理速度对于开发的影响，推而广之，找到了一些我们在做系统设计的时候需要熟知的一些数据。

首先处理器的处理速度和内存的处理速度是差距很大的，处理器的处理速度的增长速度要比内存的快很多。

![处理器与内存的性能表现.png](https://i.loli.net/2020/02/21/s8h6GTfi1PSYwpe.png)

我们需要探究的是CPU从内存中随机提取数据以及获取连续数据的速度，这是很粗略的估计，只是希望能够有一个数量级上的感知。

    Latency Comparison Numbers (~2012)
    ----------------------------------
    L1 cache reference                           0.5 ns
    Branch mispredict                            5   ns
    L2 cache reference                           7   ns                      14x L1 cache
    Mutex lock/unlock                           25   ns
    Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
    Compress 1K bytes with Zippy             3,000   ns        3 us
    Send 1K bytes over 1 Gbps network       10,000   ns       10 us
    Read 4K randomly from SSD*             150,000   ns      150 us          ~1GB/sec SSD
    Read 1 MB sequentially from memory     250,000   ns      250 us
    Round trip within same datacenter      500,000   ns      500 us
    Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
    Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
    Read 1 MB sequentially from disk    20,000,000   ns   20,000 us   20 ms  80x memory, 20X SSD
    Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms



根据2020年StackOverflow上的回答，我们可以看到Core i7 Xeon 5500 的benchmark数据如下

    Core i7 Xeon 5500 Series Data Source Latency (approximate)               [Pg. 22]
    
    local  L1 CACHE hit,                              ~4 cycles (   2.1 -  1.2 ns )
    local  L2 CACHE hit,                             ~10 cycles (   5.3 -  3.0 ns )
    local  L3 CACHE hit, line unshared               ~40 cycles (  21.4 - 12.0 ns )
    local  L3 CACHE hit, shared line in another core ~65 cycles (  34.8 - 19.5 ns )
    local  L3 CACHE hit, modified in another core    ~75 cycles (  40.2 - 22.5 ns )
    
    remote L3 CACHE (Ref: Fig.1 [Pg. 5])        ~100-300 cycles ( 160.7 - 30.0 ns )
    
    local  DRAM                                                   ~60 ns
    remote DRAM                                                  ~100 ns

而现在的cache的大小，根据wikiChip上的数据，对于Core i7-8700K

    Memory Bandwidth: 39.74 gigabytes per second
    L1 cache: 192 kilobytes (32 KB per core)
    L2 cache: 1.5 megabytes (256 KB per core)
    L3 cache: 12 megabytes  (shared; 2 MB per core)

# Reference
1. https://www.forrestthewoods.com/blog/memory-bandwidth-napkin-math/?
2. https://stackoverflow.com/questions/4087280/approximate-cost-to-access-various-caches-and-main-memory
3. https://en.wikichip.org/wiki/intel/core_i7/i7-8700k
4. https://gist.github.com/jboner/2841832