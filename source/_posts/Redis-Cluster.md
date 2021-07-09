---
title: Redis Cluster
date: 2021-07-05 17:08:09
categories:
tags:
top:
---
# Redis Cluster

# 1. Overview

- Redis集群是Redis提供的分布式数据库方案，集群通过分片 - sharding来进行数据共享，并提供复制和故障转移功能

# 2. 节点

- 概述
    - 一个redis集群通常由多个节点 node 组成
    - 开始的时候每个节点都是相互独立的，相当于各自在自己的集群当中，我们需要将各个独立的节点连接起来，构成一个包含多个节点的集群
    - `CLUSTER MEET <ip> <port>` 我们可以用这个指令来连接各个节点
        - 以节点A 收到命令，要加节点B为例
            - A和B 握手，确认彼此存在
            - A为节点B创建一个clusterNode结构，并添加到自己的clusterState.nodes字典里面
            - A根据IP 还有端口，向B发送MEET消息
            - 节点B 收到信息，为A创建clusterNode结构，并添加到自己的 `clustState.nodes` 字典里面
            - B向A发出PONG消息，让A知道自己成功接收到了
            - A向B发出PING消息
            - 节点B收到，确认A收到了自己的PONG  整个过程结束
- 启动节点
    - Redis服务器根据 `cluster-enabled` 配置选项判断是否开启服务器的集群模式
- 集群模式和standalone模式的节点区别
    - 相同之处
        - 单机模式下的功能照旧
            - 文件事件处理器
            - 时间事件处理器
            - 持久化
            - Pubsub
            - 复制模块
            - lua脚本
    - 不同点
        - 集群模式下的数据会被保存到clusterNode, clusterLink以及clusterState结构里面
            - 下面是三个结构的代码实现
            - 以及一个有三个节点的clusterState结构

```jsx
struct clusterNode {

	mstime_t ctime;

	char name[REDIS_CLUSTER_NAMELEN];

	// 记录节点角色（主/从); 记录节点目前所处状态(在线/ 下线)
	int flags;

	// 节点当前配置记录
	unit64_t configEpoch;

	char ip[REDIS_IP_STR_LEN];

	int port;

	// 保存连接节点需要的信息
	clusterLink *link;

	// ....

}
```

```jsx
typedef struct clusterLink {

	mstime_t ctime;

	// TCP 套接字描述符
	int fd;

	// 输出缓冲区，保存等待要发送给其他节点的消息
	sds sndbuf;

	// 输入缓冲区，保存着从其他节点接收到的消息
	sds rcvbuf;

	// 与这个连接相关联的节点
	struct clusterNode *node;

	// ....
} clusterLink; 
```

```jsx
typedef struct clusterState {

	clustNode *myself;

	unit64_t currentEpoch;

	// 集群当前状态
	int state;

	// 集群中至少处理着一个槽的节点的数量
	int size;

	// 集群节点名单
	dict *nodes;

	// ....
} clusterState; 
```

![clusterState示意图](https://i.loli.net/2021/07/06/XxiOA9Qjz763K1Y.png)

# 3. 槽指派

- Redis集群通过分片的方式保存数据库当中的键值对
    - 整个数据库被分为了16384个slot
    - 每个键都属于这些槽其中之一
    - 集群当中的每个节点可以处理0个或者最多16384个槽
    - 当16383个槽都有节点在处理的时候，集群处在上线状态
    - 反之，如果有任何一个槽没有得到处理，那么集群就在下线状态
- 如何指派槽
    - `CLUSTER ADDSLOTS <slot> [slot...]`

## 3.1 如何记录槽指派的信息

- clusterNode结构里面有slots属性和numslot属性
    - slots
        - bit array
        - 根据索引i上的二进制位的值来判断节点是否负责处理槽i

            ![用bit array记录槽指派信息](https://i.loli.net/2021/07/06/Zthejck4Xz2bPKl.png)

```jsx
struct clusterNode {

	// ...

	unsigned char slots[16384/8];

	int numslots; 
}
```

## 3.2 如何传播 槽指派的信息

- 一个节点除了要记录自己负责处理的槽以外，还要将自己的slots通过消息发送给集群中的其他节点，以此告知其他结点自己目前负责的槽位
- 而后槽信息会被存储在clusterState 里面
    - 指向的是一个clusterNode结构！
- 这里对比 `clusterState.slots` 以及 `clusterNode.slots` ， 前者记录了所有槽的指派信息，后者记录了当前的clusterNode结构所代表的节点的槽指派信息  一个记录了所有节点，一个记录了部分信息

```jsx
typedef struct clusterState {

	// ...

	clusterNode *slots[16384];

	// ... 
}
```

![槽指派信息的传播](https://i.loli.net/2021/07/06/4n1tgixR2NXOHdr.png)

# 4. 在集群中执行命令

## 4.1 发送指令过程

- 当客户端向节点发送和数据库键有关的命令的时候，接收命令的节点会计算出命令要处理的数据库键属于哪一个槽
- 检查这个槽是否指派给了自己
    - 如果正好指派了，节点直接执行这个命令
    - 如果没有，节点向客户端发送一个MOVED错误，指引客户端转向正确的节点，并再次发送之前想要执行的指令

        ![moved执行逻辑](https://i.loli.net/2021/07/06/MamzjpOhCXvwNI4.png)

        ![节点返回MOVED,IP以及端口号](https://i.loli.net/2021/07/06/3rBPwlGVoITAJn7.png)

        ![节点转向](https://i.loli.net/2021/07/06/zIOFRiGWZutUce9.png)

## 4.2 slots_to_keys跳跃表

- 当有新的键值对增删的操作时，节点会用clusterState结构中的slots_to_keys跳跃表来保存槽和键之间的关系

```jsx
typedef struct clusterState {

	// ...

	zskiplist *slots_to_keys;

	// ...
} clusterState; 
```

# 5. 重新分片操作

## 5.1 重分片流程

- 将任意数量已经指派给某个节点的槽改为指派给另外一个节点，相关槽所属的键值对也会从源节点被移动到目标节点

    ![重分片](https://i.loli.net/2021/07/06/t5wivjMnQuIT2rG.png)

## 5.2 ASK错误

- 发生了重分片的过程当中，一部分键已经转移到了目标节点，另外一部分还在源节点
- 这种情况下，当接收到请求之后
    - 源节点首先在自己的数据库里面查找指定的键，如果找到，直接执行
    - 如果没有找到，源节点会向客户端发送一个ASK，指引客户端转向正在导入槽的目标节点，并再次发送之前想要执行的命令

![ASK判断逻辑](https://i.loli.net/2021/07/06/nD7Ybyk1BHGzOC4.png)

# 6. 复制与故障转移

- Redis集群当中的节点分为主节点还有从节点
- 主节点用于处理槽，而从节点用于复制某个主节点，并在被复制的主节点下线以后，代替下线主节点继续处理命令请求

- 通过 `CLUSTER REPLICATE <node_id>` 来让接受命令的节点成为从节点，并开始对主节点进行复制
    - 从节点会在自己的 clusterState.nodes字典里 找到主节点对象的clusterNode结构，并将自己的 `clusterState.myself.slaveof`指针指向这个结构，以此来记录这个节点正在复制的主节点
    - 而后修改在 `clusterStat.myslef.flags`中的属性  关闭原来的REDIS_NODE_MASTER标识，打开REDIS_NODE_SLAVE标识，表示这个节点已经由原来的主节点变成了从节点
    - 根据 `clusterState.myself.slaveof`指向的clusterNode结构的IP还有端口号，开始对主节点的复制工作
- 故障检测
    - 集群中每个节点会定期向其他节点发送PING消息，以此检测对方是否在线
    - 如果接收PING消息的节点没有在规定时间内，返回PONG消息，那么没发回消息的节点会被标记为probable fail
    - 集群中的各个节点会通过互相发送消息的方式来交换集群中各个节点的状态信息
        - 在线
        - 疑似下线状态 PFAIL
        - 已下线状态 FAIL
    - 主节点会记录各个节点的报告如果半数以上的负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点会被标记为已下线
    - 该消息会向集群广播出去
- 故障转移
    - 在下线的主节点的从节点里面，选出一个从节点
        - 通过选举产生
        - 当从节点发现主节点进入已下线状态的时候，从节点会向集群广播一条 `CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST` 消息，要求所有收到这条消息并且有投票权的主节点向这个从节点投票
        - 如果一个主节点有投票权(负责处理槽) ，且还未投票给其他从节点，那么就会向要求投票的从节点返回一条 `CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK`
        - 每个参与选举的从节点统计收获到的ACK的数量
        - 具有投票权的主节点数量为N  那么从节点需要收集到大于等于 N/2 +1 张支持票来获得主节点的支持
        - 如果没有节点获得大于等于 N/2 +1 张支持票，那么就重新开始选举
    - 被选中的从节点会执行SLAVEOF no one命令，成为新的主节点
    - 新主节点会撤销所有对已下线的主节点的槽指派，并将这些槽全部指派给自己
    - 新的主节点向集群广播一条PONG消息，使得集群中的其他结点知道这个节点已经变成了主节点

# 7. 消息

节点发送的消息主要有以下5种： 

- MEET
    - 要求接受者加入到发送者当前所处的集群当中
- PING
    - 集群里的每个节点默认每隔一秒从已知节点列表中随机挑选5个结点
    - 然后对5个当中最长时间没有发送过PING消息的节点发送PING消息
- PONG
    - 来告知发送者成功收到了MEET或者PING消息
    - 也可以通过集群广播PONG 来让其他节点刷新对自己的认知
- FAIL
    - 当主节点A判断另一个主节点B进入FAIL状态了以后
    - 节点A会向集群广播一条关于节点B FAIL的消息
    - 所有收到消息的节点会将节点B标记为已下线
- PUBLISH
    - 当节点接到一个PUBLISH命令的时候，节点会执行，并向集群广播一条PUBLISH消息
    - 所有接收到的节点都会执行相同的PUBLISH命令

# 8. Redis 集群扩展与调优

> 实际案例： 5000万个键值对，每个键值对512B, 如何选择云主机的内存容量？

- 占用的内存空间
    - 50MM x 512B = 25GB
- 使用32GB主机，但是发现Redis响应有时候会非常慢
    - 查询latest_fork_usec 指标，发现快到秒级别了
- Redis持久化机制
    - RDB持久化过程当中，Redis会fork子进程来完成
    - fork用时和Redis的数据量正相关
    - fork在执行时会阻塞主线程
        - 数据量越大，fork操作造成的主线程阻塞时间就会越长
        - 当我们对25GB的数据进行持久化的时候，数据量比较大，后台运行的子进程在fork创建时阻塞了主线程，导致Redis响应变慢

- 在上述的例子里面，使用单一redis instance已经不够了，因为我们要通过数据切片来实现

## 8.1 如何保存更多的数据

- 纵向扩展
    - 升级单个Redis实例的资源配置
        - 增加内存容量
        - 增加磁盘容量
        - 使用更高配置的CPU
- 横向扩展
    - 横向增加当前Redis实例的个数

## 8.2 数据切片和实例的对象分布关系

- Redis cluster采用哈希槽来处理数据和实例之间的映射关系
    - 一个切片集群有16384个哈希槽
    - 每个键值都会根据它的key，被映射到一个哈希槽当中
- 具体映射过程
    - 根据键值对的key，按照CRC16算法计算一个16bit的值
    - 用这个16bit的值对16384取模
- reids如何初始化各个实例来分配这些槽的？
    - 部署的时候，会用到cluster create命令创建集群
    - Redis会在此时将这些槽平均分别在集群实例当中

## 8.3 客户端如何定位数据的

- 客户端定位键值对的时候，client library是可以计算出哈希槽是哪个的，但是如何定位到在哪个实例上是个问题
    - 拿到实例发过来的信息，缓存到本地上
- 最开始
    - 每个实例都只知道自己被分配了哪些哈希槽的信息，并不知道其他实例拥有的哈希槽信息
- 客户端和集群实例建立连接后
    - 实例会把哈希槽的分配信息发回给客户端
    - 各个Redis实例会互相连通，完成哈希槽信息的扩散

- Redis Cluster的重定向机制
    - 为什么需要
        - 客户端的缓存没有拿到最新的信息
            - 集群中实例有新增或者删除
            - Redis 重新分配哈希槽
    - 如何实现的
        - 客户端给一个实例发送读写操作，但是这个实例没有相应的数据
        - 实例发回一个MOVED响应，包括了新实例的访问地址

Redis使用集群方案就是为了解决单个节点数据量大、写入量大产生的性能瓶颈的问题。多个节点组成一个集群，可以提高集群的性能和可靠性，但随之而来的就是集群的管理问题，最核心问题有2个：请求路由、数据迁移（扩容/缩容/数据平衡）。

1、请求路由：一般都是采用哈希槽的映射关系表找到指定节点，然后在这个节点上操作的方案。

Redis
 
Cluster在每个节点记录完整的映射关系(便于纠正客户端的错误路由请求)，同时也发给客户端让客户端缓存一份，便于客户端直接找到指定节点，客户端与服务端配合完成数据的路由，这需要业务在使用Redis
 Cluster时，必须升级为集群版的SDK才支持客户端和服务端的协议交互。

其他Redis集群化方案例如Twemproxy、Codis都是中心化模式（增加Proxy层），客户端通过Proxy对整个集群进行操作，Proxy后面可以挂N多个Redis实例，Proxy层维护了路由的转发逻辑。操作Proxy就像是操作一个普通Redis一样，客户端也不需要更换SDK，而Redis
 Cluster是把这些路由逻辑做在了SDK中。当然，增加一层Proxy也会带来一定的性能损耗。

2、数据迁移：当集群节点不足以支撑业务需求时，就需要扩容节点，扩容就意味着节点之间的数据需要做迁移，而迁移过程中是否会影响到业务，这也是判定一个集群方案是否成熟的标准。

Twemproxy不支持在线扩容，它只解决了请求路由的问题，扩容时需要停机做数据重新分配。而Redis
 
Cluster和Codis都做到了在线扩容（不影响业务或对业务的影响非常小），重点就是在数据迁移过程中，客户端对于正在迁移的key进行操作时，集群如何处理？还要保证响应正确的结果？

Redis
 
Cluster和Codis都需要服务端和客户端/Proxy层互相配合，迁移过程中，服务端针对正在迁移的key，需要让客户端或Proxy去新节点访问（重定向），这个过程就是为了保证业务在访问这些key时依旧不受影响，而且可以得到正确的结果。由于重定向的存在，所以这个期间的访问延迟会变大。等迁移完成之后，Redis
 
Cluster每个节点会更新路由映射表，同时也会让客户端感知到，更新客户端缓存。Codis会在Proxy层更新路由表，客户端在整个过程中无感知。

除了访问正确的节点之外，数据迁移过程中还需要解决异常情况（迁移超时、迁移失败）、性能问题（如何让数据迁移更快、bigkey如何处理），这个过程中的细节也很多。

Redis Cluster的数据迁移是同步的，迁移一个key会同时阻塞源节点和目标节点，迁移过程中会有性能问题。而Codis提供了异步迁移数据的方案，迁移速度更快，对性能影响最小，当然，实现方案也比较复杂。

# Reference

1. 极客时间-Redis核心技术与实战