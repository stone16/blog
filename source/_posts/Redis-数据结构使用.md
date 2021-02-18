---
title: Redis 数据结构使用
date: 2021-02-08 20:30:04
categories: 数据存储
tags:
top:
---
# Redis 数据结构使用

# 1. String的内存空间消耗问题

## 1.1 String在保存数据时内存空间消耗较多

- String类型除了实际记录的数据，还需要额外的内存空间记录数据长度，空间使用等信息，这些信息被称为元数据
- 当实际保存的数据比较小的时候，元数据的空间开销就会比较大
- 当保存64位有符号整数的时候
    - String类型会将其保存为一个8字节的Long类型整数
    - int编码方式
- 当保存的数据中包含字符的时候
    - String 用简单动态字符串 Simple Dynamic String ， 共有三部分组成
        - buf
            - 字节数组，保存实际数据
            - 为了表示字节数组的结束，会在数组最后加一个\0. 这里会额外占用一个字节的开销
        - len
            - 占四个字节，表示buf的已用长度
        - alloc
            - 占四个字节，表示buf的实际分配长度，一般来说会大于len
    - 故而在上述的分析当中，len 和alloc就是元数据，带来了一部分的额外开销

## 1.2 RedisObject 的结构

- Redis本身支持多种数据类型，而不同数据类型都会有一些相同的元数据需要记录
    - 最后一次访问的时间
    - 被引用的次数等
- 因此Redis会用一个RedisObject结构体来统一记录这些元数据，同时指向实际数据
- 另外出于节省内存空间的考虑
    - 当保存的是Long类型整数时，RedisObject中的指针就直接赋值为整数数据了，这样就不用额外的指针再指向整数，节省了指针的空间开销
    - 当保存的是字符串数据，并且字符串小于等于44个字节，RedisObject中的元数据，指针的SDS是一块连续的内存区域，来避免内存碎片
    - 当保存的数据量大于44字节的时候，SDS的数据量就会变多，Redis就不再把SDS和RedisObject布局在一起了，会给SDS分配独立的空间，并且用指针指向SDS结构

    ![Redis Object](https://i.loli.net/2021/02/09/JoY9Hi8NqIElBDW.png)

- 在计算总共消耗的内存的时候，值得注意的是除了使用RedisObject本身，Redis还维护了一个全局哈希表来保存所有键值对，这个结构体有三个8字节的指针，共24字节。Redis使用的是jemalloc内存分配库，会根据申请的字节数N，找一个比N大，但是最接近N的2的幂次数作为分配的空间，来减少频繁分配的次数

![RedisObject Entity](https://i.loli.net/2021/02/09/xO2vECGYoRdl4jB.png)

# 2. 压缩列表

- 压缩列表的构成

    ![压缩列表构成](https://i.loli.net/2021/02/18/ALm8GYT2r7cRXqW.png)

    - 表头
        - zlbytes — 列表长度
        - zltail — 列表尾
        - zllen — 列表entry个数
    - 表尾
        - zlend — 列表结束
    - 表entry
        - 是连续的entry
            - 因为是挨着来进行放置的，所以不需要再使用额外的指针进行连接，就可以节省指针所占用的空间了
        - 包括以下几部分：
            - prev_len — 前一个entry的长度
            - len — 自身长度  4字节
            - encoding — 编码方式 1字节
            - content — 保存实际数据

    - Redis Hash类型底层有两种实现结构
        - 压缩列表
        - 哈希表
    - 通过阈值确定应该使用哪一种来保存数据
        - hash-max-ziplist-entries：表示用压缩列表保存时哈希集合中的最大元素个数。
        - hash-max-ziplist-value：表示用压缩列表保存时哈希集合中单个元素的最大长度。