---
title: Redis数据结构
date: 2021-07-02 11:18:24
categories:
tags:
top:
---
# Redis数据结构

# 1. Overview

Redis的快速除了基于内存的原因以外，另外一个是在其数据结构上的操作执行的很快。这里会走一遍Redis的现有数据结构，总结其各自的特点，Redis的数据类型和其底层数据结构的对应关系。

![Redis数据类型和底层数据结构对应关系](https://i.loli.net/2021/07/03/L6sH52KacFyROMf.png)

- Redis底层共有6中基本数据结构，其有着不同的和Redis的数据类型的对应
- List, Hash, Sorted Set, 以及Set都有两种实现结构

# 2. 键值之间的对应关系如何组织

## 2.1 全局哈希表

- Redis使用一个哈希表来保存所有的键值对
- 一个哈希表由多个哈希桶组成，每个哈希桶保存了键值对数据
- 哈希桶当中的entry元素这能够保存的是key和value的指针，分别指向实际的键值

![全局哈希表](https://i.loli.net/2021/07/03/tXTca4hIin1zNyH.png)

- 访问流程为
    - 计算键的哈希值
    - 访问到对应的哈希桶
    - 根据哈希桶来找到key和value的地址
- 为什么需要哈希桶？
    - 因为值的底层实现会是不一样的，这样子哈希桶所占内存空间是整齐且可控的
    - 然后通过指针来找到在内存当中的具体地址
    - 这个时候value 是可以分散的放置在内存块当中的

## 2.2 哈希表的冲突问题

- 哈希冲突不可避免，即两个key的哈希值和哈希桶计算对应关系的时候，落到了同一个哈希桶当中了
- 解决方式
    - 短期解决方案
        - 链式哈希
            - 同一个哈希桶中多个元素用一个链表来保存，依次用指针连接
            - 注意新的元素会加到头上
        - 存在的问题
            - 当哈希冲突链变得过长，查找速度会退化，无法实现O(1)了
    - 中长期解决方案
        - rehash  —→ 增加现有的哈希桶数量
            - 是逐渐增多的entry在更多的桶之间分散保存

## 2.3 哈希表Rehash策略

- 整体流程
    - 首先Redis默认有两个全局哈希表
    - 当你插入数据的时候，默认使用哈希表1， 此时哈希表2没有被分配空间
    - 随着数据增多，Redis开始执行rehash
        - 给哈希表2分配更大的空间，例如是1的两倍
        - 将哈希表1中的数据重新映射并拷贝到哈希表2当中
            - 这一步会涉及到大量的数据拷贝，会造成Redis线程阻塞
        - 释放哈希表1的空间
    - 另外， 在渐进式 rehash 执行期间， 新添加到字典的键值对一律会被保存到 ht[1] 里面， 而 ht[0] 则不再进行任何添加操作： 这一措施保证了 ht[0] 包含的键值对数量会只减不增， 并随着 rehash 操作的执行而最终变成空表

- 渐进式哈希
    - 是为了解决在做哈希表的复制过程当中，可能造成的线程阻塞问题
    - 一次处理分成多次，处理请求的时候顺带着更新的哈希表2当中
    - rehash的整体流程
        1. 为 `ht[1]` 分配空间，让字典同时持有 `ht[0]` 和 `ht[1]` 两个哈希表。
        2. 在字典中维持一个索引计数器变量 `rehashidx` ，并将它的值设置为 `0` ，表示 rehash 工作正式开始。
        3. 在 rehash 进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将 `ht[0]` 哈希表在 `rehashidx` 索引上的所有键值对 rehash 到 `ht[1]` ，当 rehash 工作完成之后，程序将 `rehashidx` 属性的值增 `1`
        4. 随着字典操作的不断执行，最终在某个时间点上，`ht[0]` 的所有键值对都会被 rehash 至 `ht[1]` ，这时程序将 `rehashidx` 属性的值设为 `1` ，表示 rehash 操作已完成。
    - 在渐进式哈希没有完成的时候，会同时所使用 `ht[0]` 还有 `ht[1]` 两个哈希表
        - delete find update等操作都会在两个哈希表上进行
        - 先在 `ht[0]` 然后再在 `ht[1]`
        - 而对于添加操作，会一律被保存到 `ht[1]`

![渐进式哈希的实现](https://i.loli.net/2021/07/03/pGzqNjfegolP7Lt.png)

# 3. 值的底层实现

## 3.1 简单动态字符串

### 3.1.1 什么是简单动态字符串？

- Simple Dynamic String — SDS
- Redis需要一个可以被修改的字符串值
    - 利用SDS来做键值
    - 也会被用来做缓冲区
        - AOF缓冲区
        - 客户端状态下的输入缓冲区
- 定义

```jsx
struct sdshrd {

	// 计算buf数组已经使用的字节的数量
	int len;

	// 记录buf数组未使用的字节的数量
	int free;

	// 字节数组，用来保存字符串
	char buf[];
}
```

![简单动态字符串的实现](https://i.loli.net/2021/07/03/ALkWVo5jDeX1fyY.png)

### 3.1.2 为什么需要SDS？

- 相比于常规string的优势在于：
    - 常数复杂度获取字符串长度
        - 有len字段
        - 设置和更新长度是在API执行过程当中自动完成的
    - 杜绝了缓冲区溢出
        - C字符串不会记录自身长度，容易造成缓冲区溢出—buffer overflow
        - 可能造成一些数据意外被修改
        - 而SDS的空间分配策略可以杜绝发生溢出的可能性
            - API会检测free标注的大小
            - 如果不够，那么就会进行扩展
    - 减少了内存重分配次数
        - 在C里面如果你做增加或者删除的操作，都会涉及内存重分配
- 空间优化策略
    - 空间预分配
        - 当需要扩展的时候，程序不仅为SDS分配修改所必须的空间，还会为SDS分配额外的未使用空间
    - 惰性空间释放
        - 缩短的时候并不立即使用内存重分配来回收缩短后多出来的字节
        - 使用free属性将这些字节的数量记录起来，并等待将来使用

## 3.2 双向链表

### 3.2.1 概述

- 链表
    - 可以提供高效的节点重排能力，
    - 顺序性的节点访问方法
- 什么时候会被使用
    - 当一个列表包含了数量比较多的元素
    - 列表当中包含的元素都是比较长的字符串

### 3.2.2 链表和链表节点的实现

```jsx
typedef struct listNode {

	struct listNode *prev;

	struct listNode *next;

	void *value;
}listNode;
```

```jsx
typedef struct list {

	listNode *head;

	listNode *tail;

	// 链表包含的节点数量
	unsigned long len;

	// 节点值复制函数
	void *(*dup)(void *ptr);

	// 节点值释放函数
	void (*free) (void *ptr);

	// 节点值对比函数
	void (*match)(void *ptr, void *key);
}
```

- 下方是由一个list结构和多个listNode结构组成的链表

    ![list + multi listNodes](https://i.loli.net/2021/07/03/uJoTHhdLkr4qlAc.png)

## 3.3 哈希表(字典)

### 3.3.1 什么是字典

- 用于保存键值对的数据结构
- Redis自己构建了字典的实现
- 哈希表当中可以有多个哈希表节点，而每个哈希表节点就保存了字典当中的一个键值对

### 3.3.2 dictht — 哈希表数组的实现

```jsx
typedef struct dictht {
	
	// 哈希表数组
	dictEntry **table;

	// 哈希表大小
	unsigned long size;

	// 哈希表大小掩码，用于计算索引值
	unsigned long sizemask;

	// 已有节点数量
	unsigned long used;
} dictht;
```

- table是一个数组，数组的每个元素都指向dictEntry结构的指针
    - 每个dictEntry结构保存着一个键值对
- size记录哈希表大小
- used记录目前已有节点的数量
- sizemask总是等于size - 1
    - 这个属性和哈希值一起决定了一个键应该被放到table数组的哪个索引上面

        ![dictht + dictEntry](https://i.loli.net/2021/07/03/3QTOZK1YwX6VyfG.png)

### 3.3.3 哈希表节点的实现

- 而哈希表节点是这么实现的
    - 值得注意的是有指向下一个节点的指针，是为了解决键的冲突问题

```jsx
typedef struct dictEntry {

	void *key;

	union {
		
		void *val;
		unit64_tu64;
		int64_ts64;
	} v;

	// 指向下一个哈希表节点
	struct dictEntry *next;
} dictEntry;
```

### 3.3.4 哈希表整体实现

```jsx
typedef struct dict {

	dictType *type;

	void *privatedata;

	dictht ht[2];

	// rehash索引  当rehash停止了以后，值为-1 
	in rehashidx;
}
```

- type和privatedata是针对不同类型的键值对，为创建多态字典而设置的
- ht是一个大小为2的数组，都是哈希表
    - 一般只用ht[0]的哈希表
    - 只有在做rehash的时候用ht[1]
- rehashidx 记录当前的进度
- hashfunction
    - 使用MurmurHash2算法
        - 特点
            - 即使输入的键是有规律的，算法仍能给出一个很好的随机分布性
            - 算法的计算速度也非常快

## 3.4 跳表

### 3.4.1 什么是跳表

- 为了解决链表的不足的
    - 有序链表只能逐一查找元素，导致操作起来非常缓慢
- 跳表
    - 在链表的基础上增加了多级索引，通过索引位置的跳转，实现数据的快速定位
    - 是有序集合的底层实现之一
    - 支持平均O(logN), 最坏O(N)复杂度的节点查找，还可以通过顺序性操作来批量处理节点

![跳表的实现](https://i.loli.net/2021/07/03/YgFw2MkjmpSvf7P.png)

### 3.4.2 跳表的实现

- 由两大部分构成
    - zskiplist 用来保存跳跃表节点的相关信息
        - 下图最左侧的方框
        - 属性包括
            - header — 指向跳跃表的表头节点
            - tail — 指向跳跃表的表尾节点
            - level — 记录当前跳跃表内，层数最大的那个节点的层数
            - length — 记录跳跃表的长度
    - zskiplistNode用于表示跳跃表单个节点
        - 属性包括
            - level
                - 前进指针
                    - 用于访问其他结点
                - 跨度
                    - 记录了前进指针所指向节点和当前节点的距离
                    - 累加可以用来记录某个节点在跳表当中的排位
            - backward 后退指针
                - 指向位于当前节点的前一个节点
                - 后退指针在程序从表尾向表头遍历的时候使用
            - score
            - obj 成员对象
                - 是一个指针，指向一个字符串对象，

        ![跳表实现](https://i.loli.net/2021/07/03/blB24Gdk5MzXIhH.png)

```jsx
typedef struct zskiplistNode {

	struct zskiplistLevel {

		struct zskiplistNode *forward; 

		unsigned int span;
	} level[];

	struct zskiplistNode *backward;

	double score;

	robj *obj;
} zskiplistNode;
```

```jsx
typedef struct zskiplist {

	struct zskiplistNode *header, *tail;

	unsigned long length; 

	int level;
}
```

## 3.5 整数数组

### 3.5.1 什么是整数数组

- 是set的底层实现之一，当一个集合只包含整数值元素，并且这个集合元素数量不多，Redis就会用整数集合作为集合键的底层实现

### 3.5.2 整数数组的实现

```jsx
typedef struct intset {

	uinit32_t encoding;

	uinit32_t length;

  int8_t contents[];
}
```

- encoding
    - 记录编码方式
        - INTSET_ENC_INT16
        - INTSET_ENC_INT32
        - INTSET_ENC_INT64
- length
    - 记录集合包含的元素数量
- contents
    - 数组，是其底层实现
    - 整数数组的每个原色都是contents数组的一个item
    - 各个项在数组中按照值的大小从小到大有序排列，数组中不包含任何重复项

### 3.5.3 整数数组的升级和降级

- 什么时候需要升级？
    - 当添加一个新元素，且新元素的类型比整数集合现有的所有元素的类型都要长的时候
- 如何进行升级
    - 根据新元素的类型，扩展整数集合底层数组的空间大小，并且为新元素分配空间
    - 将底层数组现有所有元素都转换成和新元素相同的类型，并将类型转换后的元素放置到正确的位上
    - 放置过程需要保证顺序不变
    - 新元素添加到底层数组当中
- 整数数组不支持降级！

## 3.6 压缩列表

### 3.6.1 什么是压缩列表

- 类似一个数组
    - 每一个元素都对应保存一个数据
    - 表头有三个字段
        - zlbytes — 列表长度，整个压缩列表占用的字节数
        - zltail — 列表尾的偏移量
        - zllen — 列表中entry个数
    - 表尾一个字段
        - zlend — 表示列表的结束
- 压缩列表是Redis为了节约内存而开发的，由一系列特殊编码的连续内存块组成的顺序型数据结构
    - 一个压缩列表可以包含任意多个节点
    - 每个节点可以保存一个字节数组或者一个整数值
    - 比起双端链表来说，更节约内存，而且内存当中连续的保存的方式，能更快的载入到缓存当中
    - 随着列表对象包含的元素越来越多，使用压缩列表的优势会逐渐消失，对象就会从底层的压缩列表的实现转向双端链表上面

### 3.6.2 压缩列表的实现

- 压缩列表节点的构成
    - previous_entry_length
        - 字节为单位
        - 记录压缩列表中前一个节点的长度
        - 可以借此通过指针运算，从当前节点的起始地址来计算出前一个节点的起始地址
        - 压缩列表的从表尾向表头遍历的操作就是使用这一原理实现的

            ![压缩列表从尾端向表头的遍历](https://i.loli.net/2021/07/03/743T9ocpMWeksxH.png)

    - encoding
        - 记录了节点的content属性所保存的数据类型和长度
    - content
        - 负责保存节点的值
            - 可以使一个字节数组或者一个整数

### 3.6.3 压缩列表的连锁更新

- 假设一个原先节点长度小于254字节的增长大于了
- 那么原先用一个字节保存的previous_entry_length已然无法保存新的数据了，那么就需要对压缩列表执行空间重分配操作
- 连锁更新，最坏情况要对压缩列表执行N次空间重分配操作，而每次空间重分配的最坏复杂度为O(N)  所以连锁更新的最坏复杂度为O(N^2)
- 连锁更新出现概率比较低，只有在压缩列表当中恰好有多个连续，长度介于250 和253字节之间的节点的时候

## 3.7 对象

- 用对象来代表Redis对于底层数据结构的上层抽象
    - 字符串
    - 双向链表
    - 哈希表
    - 集合
    - 有序集合
- 好处
    - 针对不同使用场景，为对象设置多种不同的底层数据结构实现，从而优化对象在不同场景下的使用效率
    - Redis对象系统实现了基于引用计数技术的内存回收机制
    - 当程序不再使用某个对象的时候，这个对象所占用的内存就会被自动释放

### 3.7.1 对象的实现

- 键和值都是对象，都由redisObject结构来实现的
    - type  用于记录对象的类型
        - REDIS_STRING
        - REDIS_LIST
        REDIT_HASH
        - REDIS_SET
        - REDIS_ZSET
    - encoding 记录对象所使用的编码
        - 即对象试用了什么数据结构的底层实现

```jsx
typedef struct redisObject {

	unsigned type:4;

	unsigned encoding:4;

	void *ptr;
} robj;
```

### 3.7.2 字符串对象

- 编码可以是
    - int
    - raw
        - 当保存一个字符串值，且长度大于32字节

            ![字符串对象的raw编码方式](https://i.loli.net/2021/07/03/gbGY6fzD3rF7okB.png)

    - embstr
        - 当字符串值的长度小于等于32字节的时候，会用embstr编码方式
        - embstr介绍
            - 专门用于保存短字符串的优化编码方式
            - 通过调用一次内存分配函数来分配一块连续空间，依次包含redisObject和sdshdr两个结构

                ![字符串对象embstr编码方式](https://i.loli.net/2021/07/03/G6ozCbK1UkYRO4T.png)

- 编码类型会在需要的时候进行自动的转换

### 3.7.3 列表对象

- 列表对象的编码方式是
    - ziplist — 使用压缩列表来实现

        ![列表对象的压缩列表编码方式](https://i.loli.net/2021/07/03/DHqScpNEFLiCxO7.png)

    - linkedlist — 使用双向链表来实现

        ![列表对象的双向链表编码方式](https://i.loli.net/2021/07/03/zndxC5jTAhNY7Ek.png)

- 注意：在上面的示意图当中，StringObject是做了简化的，相当于我在链表或者压缩列表对象里面嵌入了字符串对象，字符串本身是用的embstr编码方式实现的一个redisObject

- 编码转化规则
    - 使用ziplist的条件
        - 列表对象保存的所有字符串元素的长度都小于64字节
        - 列表对象保存的元素数量小于512个
    - 条件是可以进行改变的，通过改变config paras:
        - `list-max-ziplist-value`
        - `list-max-ziplist-entries`

### 3.7.4 哈希对象

- 编码方式
    - ziplist
        - 每当加新的键值对的时候，会将保存了键的压缩列表节点推入到压缩列表表尾，然后再将保存了值的压缩列表节点推入到压缩列表表尾
            - 保存了同一键值对的两个节点总是紧挨在一起
            - 值得注意  用ziplist 来实现哈希，查找是无法做到O(1)的，但同时因为值少，所以影响不会很大  [https://blog.csdn.net/zhoucheng05_13/article/details/79864568](https://blog.csdn.net/zhoucheng05_13/article/details/79864568)
            - 保存键的节点在前，保存值的节点在后

                ![哈希对象的压缩列表编码方式](https://i.loli.net/2021/07/03/bOavAyDF7u2KjhU.png)

    - hashtable

        ![哈希对象的hashtable编码方式](https://i.loli.net/2021/07/03/5AzFotmNTxdLySB.png)

- 编码转换条件
    - 使用ziplist的条件
        - 所有键值对键值字符串长度都小于64字节
        - 键值对数量小于512个
    - 修改阈值方式
        - `hash-max-ziplist-value`
        - `hash-max-ziplist-entries`

### 3.7.5 集合对象

- 编码方式有
    - intset
        - 使用整数集合作为底层实现
    - hashtable
        - 使用自定作为底层实现

- 编码转换
    - 使用intset的条件
        - 集合对象保存的所有元素都是整数值
        - 集合对象保存的元素数量不超过512个

### 3.7.6 有序集合对象

- 编码方式
    - ziplist
        - 每个集合元素用两个紧挨在一起的压缩列表节点来进行保存
        - 第一个结点保存member
        - 第二个节点保存score

            ![有序集合对象的压缩列表编码方式](https://i.loli.net/2021/07/03/1KvRnXyzwk54eLV.png)

    - skiplist
        - 包括一个字典还有一个跳跃表
            - zs1 按照分值从小到大保存了所有集合元素
                - 跳跃表节点的object属性保存了元素的成员
                - 通过跳跃表 就可以对有序集合进行范围性操作了
            - 而字典创建了成员到分值的映射
                - 可以实现O(1)复杂度查找给定成员的分值

        ```jsx
        typedef struct zset {

        	zskiplist *zs1;

        	dict *dict;
        } zset;
        ```

        ![有序集合的跳表编码方式](https://i.loli.net/2021/07/03/KeBGYXIasS4JL71.png)

- 编码转换
    - 使用ziplist的条件
        - 元素数量小于128个
        - 有序集合保存的元素成员长度小于64字节

# 4. 内存和对象的生命周期

## 4.1 类型检查

- 有些指令需要底层的特定的数据结构的实现，所以需要进行类型检查
- 通过redisObject结构的type属性来实现

    ![LLEN类型检查逻辑](https://i.loli.net/2021/07/03/F45sjTVhqWSaQfH.png)

## 4.2 内存回收

- C语言不具备自动内存回收功能，所以Redis在自己的对象系统当中构建了一个引用计数 reference counting技术实现的内存回收机制
- 通过这一机制，程序可以通过跟踪对象的引用计数信息，在适当的时候自动释放对象进行内存回收
- 引用计数信息变化的方式
    - 创建对象，被初始化为1
    - 当对象被一个新程序使用，+1
    - 当对象不再被一个程序使用，-1
    - 当引用计数值变为0，释放内存

## 4.3 对象共享

- highlight！ 只对包含整数值的字符串对象来进行共享，因为其他的类型的对象验证操作的时间复杂度比较高
- 通过引用计数来实现
- 如果键A使用值100，键B也要使用同样的值
- 那么键B的值指针会指向现在有的键A的值对象
- 然后共享的值对象的引用计数+1

    ![SDS整数值对象共享](https://i.loli.net/2021/07/03/plCK4RXTqOrID8W.png)

## 4.4 对象的空转时长

- redisObject有个lru属性，来记录这个对象上次被访问的timestamp
- 如果服务器占用的内存数超过了设定的maxmemory选项，那么空转时间比较长的那部分键就会优先被服务器释放，从而回收内存

# Reference

1. 极客时间 — Redis核心技术与实战
2. 《Redis设计与实现》
3. [http://redisbook.com/preview/dict/incremental_rehashing.html](http://redisbook.com/preview/dict/incremental_rehashing.html) 
4. [https://blog.csdn.net/zhoucheng05_13/article/details/79864568](https://blog.csdn.net/zhoucheng05_13/article/details/79864568)