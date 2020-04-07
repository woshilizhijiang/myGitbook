# Redis设计与实现分析总结

## 简短动态字符串（simple dynamic string --SDS）

### redis内部应用场景

- AOP模块中AOF缓存区
- 客户端状态中输入缓冲区

### SDS结构：

**C字符串结尾默认空字符串'\0'；一个字节**

sds.h/sdshdr

```C
struct sdshdr{
  int len;
  int free;
  char buf[];
}
```

### SDS空间非配优化策略

空间预分配和惰性空间释放

#### 空间预分配

优化SDS字符串增长操作

- len < 1MB；程序增长大于free 时，分配free == len的空间；SDS实际长度 len+free+1byte;

- len > 1MB；程序增长大于free 时，分配free的空间为固定1M；SDS实际长度 len+1MB+1byte;


#### 惰性空间释放

优化SDS字符串缩短操作

**SDS删除操作，会将len释放到free，不会缩短预分配长度；即实际长度不变。**

### 二进制安全

SDS可存储音频，视频，文件以二进制的形式
C字符串只能存文件形式



## 链表

### redis内部应用场景

- 列表键、发布与订阅、慢查询、监控 底层实现为链表

### 链表结构

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





## 字典