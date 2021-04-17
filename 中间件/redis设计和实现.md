# Redis设计与实现分析总结

### 简短动态字符串（simple dynamic string --SDS）

#### redis内部应用场景

- AOP模块中AOF缓存区
- 客户端状态中输入缓冲区

#### SDS结构：

**C字符串结尾默认空字符串'\0'；一个字节**

sds.h/sdshdr

```C
struct sdshdr{
  int len;
  int free;
  char buf[];
}
```

#### SDS空间非配优化策略

空间预分配和惰性空间释放

##### 空间预分配

优化SDS字符串增长操作

- len < 1MB；程序增长大于free 时，分配free == len的空间；SDS实际长度 len+free+1byte;

- len > 1MB；程序增长大于free 时，分配free的空间为固定1M；SDS实际长度 len+1MB+1byte;


##### 惰性空间释放

优化SDS字符串缩短操作

**SDS删除操作，会将len释放到free，不会缩短预分配长度；即实际长度不变。**

#### 二进制安全

SDS可存储音频，视频，文件以二进制的形式
C字符串只能存文件形式



### 链表

#### redis内部应用场景

- **列表键(list)、发布与订阅、慢查询、监控 底层实现为链表**

##### 链表结构

adlist.h/list

```c
typedef struct list{
  listNode *head;
  listNode *tail;
  unsigned long len;
  void *(*dup)(void *ptr);
  void *(*free)(void *ptr);
  void *(*match)(void *ptr, void *key);
}
```

**链表特性：**

双端：

无环：

带表头指针和表尾指针：

带链表长度计数器：

多态：





### 字典

字典为redis基础类型（key-value）；字典使用Hash表实现

### 跳表





### 13章 客户端



### 14章 服务端

serverCron函数默认每隔100ms执行一次，负责管理服务端的资源。

```c
//为什么server.c是 1ms 版本redis6源码
initServer::
/* Create the timer callback, this is our way to process many background
     * operations incrementally, like clients timeout, eviction of unaccessed
     * expired keys and so forth. */
if (aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL) == AE_ERR) {
    serverPanic("Can't create event loop timers.");
    exit(1);
}

//ae.c
long long aeCreateTimeEvent(aeEventLoop *eventLoop, long long milliseconds,
        aeTimeProc *proc, void *clientData,
        aeEventFinalizerProc *finalizerProc)
{
    long long id = eventLoop->timeEventNextId++;
    aeTimeEvent *te;

    te = zmalloc(sizeof(*te));
    if (te == NULL) return AE_ERR;
    te->id = id;
    aeAddMillisecondsToNow(milliseconds,&te->when_sec,&te->when_ms);
    te->timeProc = proc;
    te->finalizerProc = finalizerProc;
    te->clientData = clientData;
    te->prev = NULL;
    te->next = eventLoop->timeEventHead;
    te->refcount = 0;
    if (te->next)
        te->next->prev = te;
    eventLoop->timeEventHead = te;
    return id;
}
```



### 19.事务

mutil、exec、watch、unwatch、discard

- 事务的实现：三阶段
  - 事务开始
  - 命令入队
  - 事务执行



### 20.Lua脚本



```shell
#对lua脚本求值
10.47.188.57:0>eval "return 'hello world'" 0
"hello world"
10.47.188.57:0>eval "return 1+1" 0
"2"
#将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本；返回给定 script 的 SHA1 校验和
script load "return 2*2"
"4475bfb5919b5ad16424cb50f74d4724ae833e72"
#通过 EVALSHA 命令，可以使用脚本的 SHA1 校验和来调用这个脚本
10.47.188.57:0>evalsha "4475bfb5919b5ad16424cb50f74d4724ae833e72" 0
"4"
#
```



### 21.排序

SORT



# C源码分析

### 1.源码地址

```markdown
https://github.com/antirez/redis/tree/6.0
```

### 2.redis 6功能简图

![redis多线程架构模型](D:\developer\visualworkspace\myGitbook\截图\redis多线程架构模型.png)

### 3.多线程redis设置

```shell
#同样修改redis.conf配置文件：io-threads 4
#官方有一个建议：4核的机器建议设置为2或3个线程，8核的建议设置为6个线程，线程数一定要小于机器核数。 （超过8个基本没什么意义）
```



# 题型

## 基础

### 1.基本类型有哪些？

String 、 List、Hash、Set 、 ZSet

### 2.数据持久化类型

AOF和RDB（默认）

AOF（Append Only File）:

**Redis执行的每次写命令记录到单独的日志文件中，当重启Redis会重新将持久化的日志中文件恢复数据**



**RDB （Redis Database）：按照一定的时间将内存数据以快照的形式保存到硬盘中，对应产生的数据文件为dump.rdb。通过配置文件中的save参数来定义快照的周期。**

![](..\截图\rdb加载.png)



### 3.redis 5种基本类型应用场景

![](..\截图\redis5中数据类型应用场景.png)

- String 

  ```shell
  set myname 1
  inrc myname  # 2
  inrcby myname 3 # 5
  decr myname #4
  decrby myname 2 #2
  #setnx 集合expire 生成分布式锁
  
  ```

  

- List

- Hash

- Set

- zSet

### 4.redis过期key删除策略

1. 定时删除：设置删除时，创建定时器（Timer），让定时器在key过期来临时，立即执行删除；
2. **惰性删除：放任key不管，每次查询时判断改key是否过期；**
3. **定期删除：每隔一段时间，程序对数据库进行一次check，删除过期。**

**1，3主动；2被动；redis采用惰性删除和定期删除相结合的方式**



### 5.MySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据

redis内存数据集大小上升到一定大小的时候，就会施行数据淘汰策略。

Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。



**6种淘汰策略变为8种**
![](..\截图\redis8种淘汰策略.png)

全局的键空间选择性移除

- noeviction：当内存不足以容纳新写入数据时，新写入操作会报错；**默认策略不淘汰数据**。
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。**（这个是最常用的）**
- allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
- allkeys-lfu：设置过期时间的键空间选择性移除

- volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
- volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
- volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。
- volatile-lfu

redis 4后加入lfu



### 6.如何解决 Redis 的并发竞争 Key 问题

所谓 Redis 的并发竞争 Key 的问题也就是多个系统同时对一个 key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同！

推荐一种方案：**分布式锁**（zookeeper 和 redis 都可以实现分布式锁）。（如果不存在 Redis 的并发竞争 Key 问题，不要使用分布式锁，这样会影响性能）

基于zookeeper临时有序节点可以实现的分布式锁。大致思想为：每个客户端对某个方法加锁时，在zookeeper上的与该方法对应的指定节点的目录下，生成一个唯一的瞬时有序节点。判断是否获取锁的方式很简单，只需要判断有序节点中序号最小的一个。当释放锁的时候，只需将这个瞬时节点删除即可。同时，其可以避免服务宕机导致的锁无法释放，而产生的死锁问题。完成业务流程后，删除对应的子节点释放锁。

在实践中，当然是从以可靠性为主。所以首推Zookeeper。



### 7.分布式Redis是前期做还是后期规模上来了再做好？为什么？

既然Redis是如此的轻量（单实例只使用1M内存），为防止以后的扩容，最好的办法就是一开始就启动较多实例。即便你只有一台服务器，你也可以一开始就让Redis以分布式的方式运行，使用分区，在同一台服务器上启动多个实例。

一开始就多设置几个Redis实例，例如32或者64个实例，对大多数用户来说这操作起来可能比较麻烦，但是从长久来看做这点牺牲是值得的。

这样的话，当你的数据不断增长，需要更多的Redis服务器时，你需要做的就是仅仅将Redis实例从一台服务迁移到另外一台服务器而已（而不用考虑重新分区的问题）。一旦你添加了另一台服务器，你需要将你一半的Redis实例从第一台机器迁移到第二台机器。

### 8.什么是 RedLock

Redis 官方站提出了一种权威的基于 Redis 实现分布式锁的方式名叫 *Redlock*，此种方式比原先的单节点的方法更安全。它可以保证以下特性：

1. 安全特性：互斥访问，即永远只有一个 client 能拿到锁
2. 避免死锁：最终 client 都可能拿到锁，不会出现死锁的情况，即使原本锁住某资源的 client crash 了或者出现了网络分区
3. 容错性：只要大部分 Redis 节点存活就可以正常提供服务

往期面试题汇总：[001期~150期汇总](http://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247485351&idx=2&sn=214225ab4345f4d9c562900cb42a52ba&chksm=e80db1d1df7a38c741137246bf020a5f8970f74cd03530ccc4cb2258c1ced68e66e600e9e059&scene=21#wechat_redirect)

### 9.缓存异常

#### 缓存雪崩

**缓存雪崩是指缓存同一时间大面积的失效**，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

解决方案

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
3. 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

#### 缓存穿透

缓存穿透是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。

解决方案

1. 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
2. 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
3. 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

### 10.假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？

使用keys指令可以扫出指定模式的key列表。

对方接着追问：如果这个redis正在给线上的业务提供服务，那使用keys指令会有什么问题？

这个时候你要回答redis关键的一个特性：redis的单线程的。keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，**scan指令可以无阻塞的提取出指定模式的key列表**，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

### 11.使用Redis做过异步队列吗，是如何实现的

使用**list类型保存数据信息，rpush生产消息，lpop消费消息**，当lpop没有消息时，可以sleep一段时间，然后再检查有没有信息，如果不想sleep的话，可以使用blpop, 在没有信息的时候，会一直阻塞，直到信息的到来。redis可以通过pub/sub主题订阅模式实现一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢失。

### 12.Redis如何实现延时队列

使用**sortedset，使用时间戳做score, 消息内容作为key,调用zadd来生产消息，消费者使用zrangbyscore获取n秒之前的数据做轮询处理**。

### 13.Redis回收进程如何工作的？

1. 一个客户端运行了新的命令，添加了新的数据。
2. Redis检查内存使用情况，如果大于maxmemory的限制， 则根据设定好的策略进行回收。
3. 一个新的命令被执行，等等。
4. 所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

### 14.Redis回收使用的是什么算法？

LRU算法；LFU