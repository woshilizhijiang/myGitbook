

### redis

#### 基于redis的分布式令牌桶限流器

令牌桶两种做法：

- 定时任务：定时任务问题消耗系统资源。
- 延时计算：**resync函数**。该函数会在每次获取令牌之前调用；思路：若当前**时间晚于nextFreeTicketMicros**,则计算该段时间内生成多少令牌，将生产的令牌加入令牌桶中并更新数据。这样只需获取令牌时计算一次即可。

关键变量：

- nextFreeTicketMicros：表示下一次允许补充令牌的时间。
- maxPermits：最大令牌数。
- stroredPermits：当前存储的令牌数，数量不能超过最大令牌数。

```c++
void resync(long nowMicros){
	//if nextFreeTicket is in the past,resync to now
	if(nowMicros > nextFreeTicketMicros){
		double newPermits = (nowMicros - nextFreeTiketMicros)/coolDownIntervalMicros();
		storedPermits = min(maxPermits,storedPermits + newPermits);
		nextFreeTicketMicros = nowMicros;
	}
}
```





