

### redis

#### 基于redis的分布式令牌桶限流器

令牌桶两种做法：

- 定时任务：定时任务问题消耗系统资源。
- 延时计算：**resync函数**。该函数会在每次获取令牌之前调用；思路：若当前**时间晚于nextFreeTicketMicros**,则计算该段时间内生成多少令牌，将生产的令牌加入令牌桶中并更新数据。这样只需获取令牌时计算一次即可。

关键变量：

- nextFreeTicketMicros：表示下一次允许补充令牌的时间。
- maxPermits：最大令牌数。
- stroredPermits：当前存储的令牌数，数量不能超过最大令牌数。

```c
void resync(long nowMicros){
	//if nextFreeTicket is in the past,resync to now
	if(nowMicros > nextFreeTicketMicros){
		double newPermits = (nowMicros - nextFreeTiketMicros)/coolDownIntervalMicros();
		storedPermits = min(maxPermits,storedPermits + newPermits);
		nextFreeTicketMicros = nowMicros;
	}
}
```



#### Redis单线程的高并发和快速原因

1.redis是基于内存的，内存的读写速度非常快；

2.redis是单线程的，省去了很多上下文切换线程的时间；

3.**redis使用多路复用技术**，可以处理并发的连接。非阻塞IO 内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间。

注：单线程多进程集群方案



