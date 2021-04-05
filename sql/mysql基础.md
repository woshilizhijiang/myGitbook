



## MYSQL基础



### 一、基础语句

```sql
--1.版本查询
select version(); --苏宁 5.7.19-17-log

```

#### 面试指南

1. **MySql读写IO的操作过程** 
   https://zhuanlan.zhihu.com/p/89051646
2. 



### 二、分析

#### 1.三种log

```markdown
1.重做日志(redo log)

2.回滚日志(undo log)

3.二进制日志(bin log)

```

### 2数据库连接池

```markdown
//1万并发用户访问
这个网站的数据库连接池应该设置成多小呢？
https://mp.weixin.qq.com/s/JvRh7M47qjg9hOzw8nUBCg
oracle官网9600并发，连接池2048；验证连接数2048，1024，...，96性能直线提升

//数据库连接池常用：连接数 = ((核心数 * 2) + 有效磁盘数)

//有限的资源
//数据库的性能瓶颈时，总是可以将其归为三类：CPU、磁盘、网络

```



## 数据库剖析

mysql区别与其他数据库的特点：**插件式存储引擎**

mysql体系机构

1. **连接池**
2. 管理服务和工具组件
3. sql接口组件
4. **查询分析器**
5. **性能优化组件**
6. **缓存**
7. **插件式引擎**
8. **物理文件**

#### innodb简单叙述

​	InnoDB存储引擎支持事务，主要处理OLTP方面应用；特点：**行锁设计、支持外键、读取不产生锁**；

ibd文件

- 表中数据存储：**采用聚簇索引的方式，引申出非聚簇索引**； innodb两大索引(clustered index 、non-clustered index)

  clustered index 每张表的存储按照主键顺序存放，**如果没有指定primary key；自动生成6字节 rowid 作为主键**



#### 1章 innodb存储引擎

##### **核心场景**

- OLTP应用中，innodb作为核心应用表的首选存储引擎
- OLAP应用中，MyISAM主要应用场景

#### 2章 **Innodb体系架构**



![image-20210209152549619](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210209152549619.png)

1.**innodb由多个小内存块组装成一个大内存池，负责如下工作**

- 维护所有进程/线程需要访问的多个内部数据结构
- 缓存磁盘上的数据，方便快速读取，并对磁盘文件的数据进行修改之前在这里缓存
- 重做日志（redo log）缓存

2.**后台线程的主要作用负责刷新内存池中的数据，保证缓存池中的内存缓存是最近数据。（LRU）**

- 默认情况innodb后台有7个线程：4个IO thread、1个master thread、1个锁监控、1个错误监控；io数量由配置文件中innodb_file_io_threads参数控制，默认4；
- 4个io线程：insert buffer thread、log thread、read thread、write thread；innodb plugins增加默认 IO thread数量；分为别为4，不再使用innodb_file_io_threads参数；分布为innodb_read_io_threads 、innodb_write_io_threads
  **mysql版本5.7.19-17-log** 读写线程默认分别为8个。

##### 3.**内存**

![image-20210210091432508](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210210091432508.png)

- innodb存储引擎组成：缓冲池（buffer pool）、重做日志缓冲池（redo log buffer）以及额外的内存池（additonal memory pool）
- 缓冲池：innodb存储引擎的工作方式总是将数据库文件按页（每页16K）读取到缓冲池，然后按LRU算法保留在缓存池数据。
  修改文件，先修改缓存页（修改后为脏页），按一定频率将脏页flush到文件。
- 日志缓冲：重做日志每秒刷新缓存到日志文件
- 额外内存池：内存堆的方式进行，



##### 2.3 **master thread**

master thread线程优先级最高。由几个循环组成：主循环、后台循环、刷新循环、暂停循环。

loop分两大部分操作：**每秒钟和每10秒**
master thread潜在问题：
对IO有限制，

关键特性：innodb存储引擎包含插入缓冲、两次写、自适应哈希索引（adaptive hash index）、预读；

**插入缓冲：**

针对插入性能提升

**两次写：**

数据可靠性提升
![image-20210210113543717](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210210113543717.png)

**自适应哈希索引**

**查找性能提升，理论上时间复杂度为O(1)**

**2.5启动、关闭与恢复**



#### 3章 文件

mysql数据库和innodb存储引擎表的各种类型文件

- 参数文件：mysql实例初始化参数
  mysql --help | grep my.cnf

- 日志文件：记录错误、二进制、慢查询、查询日志
  
- socket文件：用unix域套接字方式进行连接时需要的文件
  
- pid文件：进程ID
  
- MYSQL表结构文件：用来存放mysql表结构定义文件
  
- 存储引擎文件：存储引擎的各个文件类型
  **frm**
  **innodb存储引擎文件**
  表空间文件：初始化10Mb，ibdata1的文件 。.ibd结尾文件

  ![image-20210210141308983](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210210141308983.png)

  重做日志文件：

  ![image-20210210141447981](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210210141447981.png)

  

#### 4章 表

- 主要讲逻辑存储及实现；分析表的存储特性，即数据是如何组织和存放的。

##### 索引组织表



##### innodb逻辑存储结构

![image-20210220114559647](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210220114559647.png)

- 表空间：存放索引逻辑数据；由段（segment）、区(extent)、页(page,页又称块)组成

启用innodb_file_per_table，每张表的表空间内存存放的只是数据、索引和插入缓冲bitmap页，其他数据如回滚信息、插入缓冲索引页、系统事务信息、二次写缓冲都放在原来的共享表空间中。说明一个问题即使启用了参数innodb_file_per_table参数，共享表空间还是会不断地增加其大小。

- 段：数据段、索引段、回滚段等组成；
- 区：由连续的页组成，**任何情况下每个区大小都是1MB**；innodb为保证页的连续性，一次申请4~5个区； 默认情况下，innodb存储引擎页大小为16kB，即一个区一共有64个连续页；innodb1.2.x版本新增innodb_page_size，注定设定参数大小。
- 页：innodb磁盘管理的最小单位；**设置完成后无法修改**，常见页如下：
  - 数据页（B-tree node）
  - undo页（undo log page）
  - 系统页（system page）
  - 事务数据页（transaction system page）
  - 插入缓冲位图页
  - 插入缓冲空闲列表页
  - 未压缩的二级制大对象页
  - 压缩的二级制大对象页
- 行：innodb是面向列，也就是说数据是按行进行存放；每个页的行记录也是有硬性定义的，最多允许存放**16KB/2-200行的记录，即7992行记录。**
  

##### innodb行记录格式

- mysql提供两种格式记录行数据；compact、redundant（**mysql5.0及以前版本格式**,兼容低版本mysql数据而存在的）
- compact：
  ![image-20210220160305346](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210220160305346.png)

- redundant

  ![image-20210220160656538](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210220160656538.png)

- compressed和dynamic行记录格式



##### innodb数据页结构

- 页的7个组成部分

##### 约束

- 数据完整性：通过primary key 和 unique key保证；或者通过触发器、外键、选择合适数据类型确保一个数据值满足特定条件、Default、not null
- 约束的创建和查找：
  - 约束创建两种方式：表创建时进行约束，利用alter table命令进行创建约束
- 约束和索引的区别
- 对错误数据的约束
- enum和set约束
- 约束器与约束：trigger
- 外键约束

##### 视图

- **View命名虚表，由sql查询定义，可以当表使用；与持久表不同的时，视图没有实际物理存储**
- 物化视图

##### 分区表

- range分区
- list分区
- hash分区
- key分区
- 子分区

#### 5章 索引及算法

##### innoDB存储引擎索引概述

InnoDB索引支持一下几种常见的索引:

- B+树索引:传统意义上的索引,目前关系型数据库最常用和最有效的索引.
- 全文索引
- 哈希索引:自适应的,主键自适应递增

##### 数据结构和算法

二分法查找:

二叉树和平衡二叉树查找:

##### B+树

- 由B树和索引顺序访问方法演化而来
- B+树定义:待补充......简短介绍,B+树**为磁盘或者其他直接存取辅助设备设计的一种平衡查找树**。
  ![image-20210222191559305](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210222191559305.png)

- B+树的插入操作:插入后叶子节点中记录依然排序,插入B+树的三种情况

  - - | Leaf page满 | index page满 | 操作                                                         |
      | ----------- | ------------ | ------------------------------------------------------------ |
      | NO          | NO           | 直接将记录插入到叶子节点                                     |
      | YES         | NO           | 1)拆分Leaf page<br />2)将中间的节点放到index page中<br />3)小于中间节点记录放左边<br />4)大于或等于中间节点的记录放右边 |
      | YES         | YES          | 1)拆分Leaf page<br />2)小于中间节点记录放左边<br />3)大于或等于中间节点的记录放右边<br />4)拆分index page<br />5)小于中间节点的记录放左边<br />6)大于等于中间节点的记录放右边<br />7)中间节点放入上一层index page |

- B+树的删除操作

  - 使用填充因子来控制树的删除变化,50%是填充因子可设的最小值.和插入一样,删除同意考虑三种情况,不同的是,删除根据填充因子的变化来衡量

    - | 叶子节点小于fill factor | 中间节点小于fill factor | 操作                                                         |
      | ----------------------- | ----------------------- | ------------------------------------------------------------ |
      | NO                      | NO                      | 叶子节点直接删除,如果为index page,用该节点右节点代替         |
      | YES                     | NO                      | 合并叶子节点和它的兄弟节点,同时更新index page                |
      | YES                     | YES                     | 1) 合并叶子节点和它的兄弟节点<br />2) 更新index page<br />3) 合并index page和它的兄弟节点 |

  ![image-20210222195229970](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210222195229970.png)

  ![image-20210222195141297](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210222195141297.png)

  ![image-20210222195418483](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210222195418483.png)

  ![image-20210222195551426](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210222195551426.png)

##### B+树索引

- B+树索引的本质就是B+树在数据库中的实现.在数据库中有一个特点高扇出性,高度一般为2~4层,意味着查找某一键值最多只需2~4次IO.
- 数据库中B+树索引可分为聚集索引(clusterd index)和辅助索引(secondary index);两者不同是叶子节点存放的是否是一整行的信息.
- 聚集索引 clustered index
  - 叶子节点为整张表行记录数据,也称数据页.
- 辅助索引 secondary index
  - 叶子节点并不包含全部数据;
- B+树索引的分裂
- B+树索引管理
  - 索引管理
    - 创建和删除:ALTER TABLE 和 CREATE/DROP index

![image-20210223193238212](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210223193238212.png)

- Cardinality值:

##### B+**树索引的使用**

- 不同应用中B+树索引的使用
  - OLAP和OLTP
    - 联合索引：表山多列进行索引
    - 覆盖索引：
    - 优化器选择不适用索引的情况
    - 索引提示

##### 哈希算法

- 常见算法，时间复杂度O（1），
- 哈希表:
- innodb存储引擎中的哈希算法
- 自适应哈希索引

##### 全文检索：1.2.x版本开始支持

- 倒排索引实现全文索引
- innodb全文检索
- 全文检索

#### 6章 锁

锁机制保证每个用户一致的方式读取和修改数据。

##### 什么是锁

- 锁是数据库系统区别于文件系统的一个关键特性。锁机制用于管理对共享资源的并发访问。

##### lock和latch

- lock和latch都是锁，latch一般称为闩锁（轻量级的锁），要求锁时间必须非常短。若持续时间长，应用性能会非常差。innodb中latch分mutex（互斥量）和rwlock（读写锁）。目的用来保证**并发线程操作临界资源的正确性**，并且通常没有死锁检测的机制。
- lock的对象是事务，锁定数据库中的对象，如表、页、行。并且一般lock的对象仅在事务commit或rollback后进行释放。

![image-20210224210856690](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210224210856690.png)



![image-20210224211146513](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210224211146513.png)

##### innodb存储引擎中的锁

- 锁的类型
  - 共享锁（S Lock），**允许事务读一行数据**
  - 排他锁（X Lock），**允许事务删除或更新一行数据**
  - ![image-20210224211541356](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210224211541356.png)
    X、S锁都为行锁
  - **innodb支持意向锁**表示 表或行锁同时支持，
    - 意向共享锁
    - 意向排他锁
- 一致性非锁定读
  - **MVCC(Mutil Version Concurrency Control)多版本并发控制**
  - **innodb通过多版本控制的方式来读取当前执行时间数据库中行的数据，即如果读取行正在执行delete或update，读操作不会去等待行锁释放**
  - innodb使用非锁定的一致性读
    - Read commited事务隔离级别下，对于快照数据，总是读取被锁定的最新一份snapshot数据
    - repeatable read事务隔离级别下，对于快照数据，事务开始时的行数据版本
- 一致性锁定读
  - mysql提供两种一致性锁定读
    - **select ... for update  :加X锁**
    - **select ... lock in share mode ：加S锁**
- 自增长与锁
  - 主键自增长加锁，保证高并发下安全。innodb行锁考虑插入问题
  - MYISAM是表锁，粗暴简单无效考虑插入问题。
- 外键与锁
  - **外键列innodb自动加索引，oracle不自动加。**

##### 锁的算法

- 行锁的3种算法（Innodb）
  - **Record Lock：单个行记录上的锁（主键隐式的进行锁定）**
  - **Gap Lock：间隙锁，锁定一个范围，但不包含记录本身**
  - **Next-Key Lock： Gap Lock + Record Lock，锁定一个范围，并且锁定记录本身。（innodb行锁采用这种算法）**

- 解决phantom problem（幻读）
  - **innodb repeatable read采用Next-Key Lock算法阻止幻读，oracle则需要serializable事务隔离级别**

##### 锁问题

锁只会带来三种问题

- 脏读：不同事务下，当前事务可以读到另外事务未提交的数据，简单说就是可以读到脏数据。
  - 绝大多数数据库是read committed，innodb是read repeatable
  - 特殊场景read uncommited会出现脏读
- 不可重复读
  - 两个事务读取到不一样的已提交数据，违反数据库事务一致性的要求。
- 丢失更新
  - 一事务更新被另一个事务更新覆盖，导致数据不一致。
  - 当前数据库隔离级别下，理论意义上都不会导致丢失更新。

##### 阻塞

- 竞争同一个资源导致阻塞。innodb_lock_wait_timeout（默认50s）控制等待的时间。

##### 死锁

- **两个或以上事务执行过程中，争夺锁资源而造成一种相互等待的现象。**
  - **解决办法：1.将任何等待转化为回滚，并且事务重新开始**
  - **解决办法：2.超时机制**

##### 锁升级

- innodb不存在锁升级问题
- sql server有锁升级问题，行锁变页锁

#### 7章 事务

##### 认识事务

- 分类
  - 扁平：begin work， commit work，rollback work 
  - 带有保存点的扁平：savepoint 不持久化
  - 链：持久化
  - 嵌套
  - 分布式

##### 事务的实现

ACID

A、D通过redo log ，C通过undo log

##### 分布式事务

- mysql数据库分布式事务
  - ![](..\截图\分布式事务模型.png)
  - 允许多个独立的事务资源（）
  - XA（**eXtended Architecture**）事务
  - innodb使用分布式事务，事务隔离级别必须是serializable，
  - **2阶段提交(two-phase commit)**

##### 不好的事务习惯

- 在循环中提交事务
- 使用自动提交
- 使用自动回滚

##### 长事务

执行时间较长的事务。场景银行账号过亿更新，需要1小时或者4、5小时。

- 对于长事务问题，将其转化为多个小批量的事务处理。一亿用户 拆分为10个用户级别

### 8.备份和恢复

mysqldump、ibbackup、replication。第三方工具 xtrabacup、LVM快照备份等。

##### 8.1备份与恢复概述

备份方式

- hot backup 热备份
- cold backup 冷备份
- warm backup 温备份：全局锁，保证数据一致性

备份后文件内容：

- 逻辑备份
- 裸文件备份

备份数据库内容来分

- 完全备份
- 增量备份
- 日志备份






### MyISAM简单叙述

不支持事务，表锁、全文索引；对于OLAP操作速度快

- 存储引擎由MYD和MYI组成，分别存放数据和索引



### 基础操作

show engines;

![image-20210208175607005](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210208175607005.png)

alter table mytest engine=mysiam as select * from dlv_vm_info

show engine  innodb status ;

show variables like 'innodb_version'

#### sql4种标准事务隔离级别

1. read uncommited：未提交可读取
2. read commited：提交可读取(一般数据库默认)
3. reapatable read：可重复读  **(mysql innodb默认)**
4. serializable：可串行化

#### 基础语句

![](..\截图\explain2.png)

![](..\截图\optimizer_trace.png)

```sql
--1.between and 范围包含边界相当于 >= 和 <=

--2.MySQL中大于小于，IN，OR，BETWEEN性能比较
--2.1explain 性能展示相同
explain
select id from meta_host_fragment_monitor
where valid = 1 and id between 80001 and 80004;



select id from meta_host_fragment_monitor
where valid = 1
and id >= 80001 and id<= 80004;

  “ranges”: [
      “100 <= id <= 104″
  ]

select id from meta_host_fragment_monitor
where valid = 1 and id in(80001 ,80002,80003,80004);
“ranges”: [
    “100 <= id <= 100″,
    “101 <= id <= 101″,
    “102 <= id <= 102″,
    “103 <= id <= 103″,
    “104 <= id <= 104″
]

select id from meta_host_fragment_monitor
where valid = 1 and id=80001 or id = 80002 or id = 80003 or id = 80004;
“ranges”: [
    “100 <= id <= 100″,
    “101 <= id <= 101″,
    “102 <= id <= 102″,
    “103 <= id <= 103″,
    “104 <= id <= 104″
]


--2.2 OPTIMIZER_TRACE
show variables like '%optimizer_trace%';
--1.打开optimizer trace功能 (默认情况下它是关闭的): ,one_line=on可选
SET optimizer_trace="enabled=on,one_line=on"
--2.这里输入你自己的查询语句
SELECT ...;
--3.从OPTIMIZER_TRACE表中查看上一个查询的优化过程
SELECT * FROM information_schema.OPTIMIZER_TRACE;
--4.可能你还要观察其他语句执行的优化过程，重复上边的第2、3步
...
--5.当你停止查看语句的优化过程时，把optimizer trace功能关闭
SET optimizer_trace="enabled=off,one_line=off";

```



### 性能优化

#### 1.千万级别数据分页优化

```sql
-- 参考 https://www.jb51.net/article/31868.htm
--1.原始方法 
SELECT * FROM table ORDER BY id LIMIT 1000, 10; 
SELECT * FROM table ORDER BY id LIMIT 1000000, 10; --百万级以后将很慢

--2.初步优化-id索引优化
SELECT * FROM table WHERE id >= (SELECT id FROM table LIMIT 1000000, 1) LIMIT 10; 
--速度提升到0.x秒

--3.进一步优化between...and
--针对id连续的  使用场景比较少
SELECT * FROM table WHERE id BETWEEN 1000000 AND 1000010; ---比2快 5至10倍 

--4.id不连续场景
SELECT * FROM table WHERE id IN(10000, 100000, 1000000...); 

--5.注：
--查询字段一较长字符串的时候，表设计时要为该字段多加一个字段,如，存储网址的字段 
--查询的时候，不要直接查询字符串，效率低下，应该查诡该字串的crc32或md5 
```



#### 2.explain

![](..\截图\explain解释1.png)

```markdown
1. id
SELECT识别符。这是SELECT查询序列号。这个不重要,查询序号即为sql语句执行的顺序，看下面这条sql

2. select_type
select类型，它有以下几种值
2.1 simple 它表示简单的select,没有union和子查询
2.2 primary 最外面的select,在有子查询的语句中，最外面的select查询就是primary,
2.3 union union语句的第二个或者说是后面那一个.现执行一条语句
2.4 DEPENDENT UNION：一般是子查询中的第二个select语句（取决于外查询，mysql内部也有些优化）
2.5 UNION RESULT：union的结果
2.6 SUBQUERY：子查询中的第一个select
2.7 DEPENDENT SUBQUERY：子查询中第一个select，取决于外查询（在mysql中会有些优化，有些dependent会直接优化成simple）
2.8 DERIVED：派生表的select（from子句的子查询）奇怪的是在5.7的版本中竟然只有一个SIMPLE

3. table
输出的行所用的表，这个参数显而易见，容易理解。

4. type
常用的类型有： ALL、index、range、 ref、eq_ref、const、system、NULL（从左到右，性能从差到好）
ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行
index: Full Index Scan，index与ALL区别为index类型只遍历索引树
range:只检索给定范围的行，使用一个索引来选择行，一般出现在>、<、in、between
ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值；非唯一索引
eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。
如将主键置于where列表中，MySQL就能将该查询转换为一个常量，
system是const类型的特例，当查询的表只有一行的情况下，使用system
NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

5. possible_keys
显示可能应用在这张表中的索引，但不一定被查询实际使用

6. key
实际使用的索引。

7. key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。一般来说，索引长度越长表示精度越高，效率偏低；长度越短，效率高，但精度就偏低。并不是真正使用索引的长度，是个预估值。

8. ref
表示哪一列被使用了，常数表示这一列等于某个常数。

9. rows
大致找到所需记录需要读取的行数。

10. filter
表示选取的行和读取的行的百分比，100表示选取了100%，80表示读取了80%。

11. extra
一些重要的额外信息

Using filesort：使用外部的索引排序，而不是按照表内的索引顺序进行读取。（一般需要优化）
Using temporary：使用了临时表保存中间结果。常见于排序order by和分组查询group by（最好优化）
Using index：表示select语句中使用了覆盖索引，直接冲索引中取值，而不需要回行（从磁盘中取数据）
Using where：使用了where过滤
Using index condition：5.6之后新增的，表示查询的列有非索引的列，先判断索引的条件，以减少磁盘的IO
Using join buffer：使用了连接缓存
impossible where：where子句的值总是false
还有一些，基本上很少遇到，就不作说明了。
```



## 数据库问题

### 1.MySQL 的事务隔离级别有哪些？分别用于解决什么问题？

主要用于**解决脏读、不可重复读、幻读**。

脏读：**一个事务读取到另一个事务还未提交的数据**。

不可重复读：**在一个事务中多次读取同一个数据时，结果出现不一致**。

幻读：**在一个事务中使用相同的 SQL 两次读取，第二次读取到了其他事务新插入的行**。

**不可重复**读注重于**数据的修改**，而**幻读**注重于**数据的插入**。

**read committed：主流数据库默认**

<div style='color:#F00'>repeatable read是Mysql默认</div>

![](..\截图\数据库隔离级别.png)

serializable：数据库层面分布式事务要求。

### 2.MySQL 的可重复读怎么实现的？？？？

**使用 MVCC 实现的，即 Mutil-Version Concurrency Control，多版本并发控制**。关于 MVCC，比较常见的说法如下，包括《高性能 MySQL》也是这么介绍的。

**InnoDB 在每行记录后面保存两个隐藏的列，分别保存了数据行的创建版本号和删除版本号**。每开始一个新的事务，系统版本号都会递增。事务开始时刻的版本号会作为事务的版本号，用来和查询到的每行记录的版本号对比。在可重复读级别下，MVCC是如何操作的：

SELECT：必须同时满足以下两个条件，才能查询到。1）**只查版本号早于当前版本的数据行；**2）**行的删除版本要么未定义，要么大于当前事务版本号。**

INSERT：为插入的每一行保存当前系统版本号作为创建版本号。

DELETE：为删除的每一行保存当前系统版本号作为删除版本号。

UPDATE：插入一条新数据，保存当前系统版本号作为创建版本号。同时保存当前系统版本号作为原来的数据行删除版本号。



**MVCC 只作用于 RC（Read Committed）和 RR（Repeatable Read）级别**，因为 **RU（Read Uncommitted）总是读取最新的数据版本，而不是符合当前事务版本的数据行。而 Serializable 则会对所有读取的行都加锁。这两种级别都不需要 MVCC 的帮助。** 

最初我也是坚信这个说法的，但是后面发现在某些场景下这个说法其实有点问题。

举个简单的例子来说：如果线程1和线程2先后开启了事务，事务版本号为1和2，如果在线程2开启事务的时候，线程1还未提交事务，则此时线程2的事务是不应该看到线程1的事务修改的内容的。

但是如果按上面的这种说法，由于线程1的事务版本早于线程2的事务版本，所以线程2的事务是可以看到线程1的事务修改内容的。

？？？？？？？
**实际上，InnoDB 会在每行记录后面增加三个隐藏字段：**

**DB_ROW_ID：行ID**，随着插入新行而单调递增**，如果有主键，则不会包含该列。**

**DB_TRX_ID：记录插入或更新该行的事务的事务ID。**

**DB_ROLL_PTR：回滚指针，指向 undo log 记录**。每次对某条记录进行改动时，该列会存一个指针，可以通过这个指针找到该记录修改前的信息 。当某条记录被多次修改时，该行记录会存在多个版本，通过DB_ROLL_PTR 链接形成一个类似版本链的概念。



 

接下来进入正题，以 RR 级别为例：每开启一个事务时，系统会给该事务会分配一个事务 Id，在该事务执行第一个 select 语句的时候，会生成一个当前时间点的事务快照 ReadView，主要包含以下几个属性：

trx_ids：生成 ReadView 时当前系统中活跃的事务 Id 列表，就是还未执行事务提交的。

up_limit_id：低水位，取 trx_ids 中最小的那个，trx_id 小于该值都能看到。

low_limit_id：高水位，生成 ReadView 时系统将要分配给下一个事务的id值，trx_id 大于等于该值都不能看到。

creator_trx_id：生成该 ReadView 的事务的事务 Id。

 

有了这个ReadView，这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见：

1）如果被访问版本的trx_id与ReadView中的creator_trx_id值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。

2）如果被访问版本的trx_id小于ReadView中的up_limit_id值，表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。

3）如果被访问版本的trx_id大于ReadView中的low_limit_id值，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。

4）如果被访问版本的trx_id属性值在ReadView的up_limit_id和low_limit_id之间，那就需要判断一下trx_id属性值是不是在trx_ids列表中。如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。

 

**在进行判断时，首先会拿记录的最新版本来比较，如果该版本无法被当前事务看到，则通过记录的 DB_ROLL_PTR 找到上一个版本，重新进行比较，直到找到一个能被当前事务看到的版本。**

 

而对于删除，其实就是一种特殊的更新，InnoDB 用一个额外的标记位 delete_bit 标识是否删除。当我们在进行判断时，会检查下 delete_bit 是否被标记，如果是，则跳过该版本，通过 DB_ROLL_PTR 拿到下一个版本进行判断。

 

以上内容是对于 RR 级别来说，而对于 RC 级别，其实整个过程几乎一样，唯一不同的是生成 ReadView 的时机，RR 级别只在事务开始时生成一次，之后一直使用该 ReadView。而 RC 级别则在每次 select 时，都会生成一个 ReadView。

### 3.那 MVCC 解决了幻读了没有？

**幻读：在一个事务中使用相同的 SQL 两次读取，第二次读取到了其他事务新插入的行，则称为发生了幻读。**

例如：

1）事务1第一次查询：select * from user where id < 10 时查到了 id = 1 的数据

2）事务2插入了 id = 2 的数据

3）事务1使用同样的语句第二次查询时，查到了 id = 1、id = 2 的数据，出现了幻读。

 

谈到幻读，首先我们要引入“**当前读**”和“**快照读**”的概念，聪明的你一定通过名字猜出来了：

**快照读**：生成一个**事务快照（ReadView）**，之后都从这个快照获取数据。**普通 select 语句就是快照读**。

**当前读：读取数据的最新版本。常见的 update/insert/delete、还有 select ... for update、select ... lock in share mode 都是当前读。**

 

对于**快照读，MVCC 因为从 ReadView 读取，所以必然不会看到新插入的行，所以天然就解决了幻读的问题。**

 

而**对于当前读的幻读，MVCC 是无法解决的。需要使用 Gap Lock 或 Next-Key Lock（Gap Lock + Record Lock）来解决**。

 

其实原理也很简单，用上面的例子稍微修改下以触发当前读：**select * from user where id < 10 for update**，当使用了 **Gap Lock 时，Gap 锁会锁住 id < 10 的整个范围，因此其他事务无法插入 id < 10 的数据，从而防止了幻读。**

### 4.那经常有人说 Repeatable Read 解决了幻读是什么情况？

**SQL 标准中规定的 RR 并不能消除幻读**，但是 **MySQL 的 RR 可以，靠的就是 Gap 锁**。**在 RR 级别下，Gap 锁是默认开启的，而在 RC 级别下，Gap 锁是关闭的**

### 5.什么是索引？

索引（Index）是帮助 MySQL 高效获取数据的**数据结构**。简单的理解，索引类似于字典里面的目录。

###  6.常见的索引类型有哪些？

常见的索引类型有：**hash、~~b树~~、b+树**。

hash：底层就是 hash 表。进行查找时，根据 key 调用hash 函数获得对应的 hashcode，根据 hashcode 找到对应的数据行地址，根据地址拿到对应的数据。

B树：B树是一种多路搜索树，n 路搜索树代表每个节点最多有 n 个子节点。每个节点存储 key + 指向下一层节点的指针+ 指向 key 数据记录的地址。查找时，从根结点向下进行查找，直到找到对应的key。

B+树：B+树是b树的变种，主要区别在于：**B+树的非叶子节点只存储 key + 指向下一层节点的指针**。另外，**B+树的叶子节点之间通过指针来连接，构成一个有序链表，因此对整棵树的遍历只需要一次线性遍历叶子结点即可**。


### 7.为什么MySQL数据库要用B+树存储索引？而不用红黑树、Hash、B树？

红黑树：如果在内存中，红黑树的查找效率比B树更高，但是涉及到磁盘操作，B树就更优了。因为**红黑树是二叉树，数据量大时树的层数很高**，从树的根结点向下寻找的过程，每读1个节点，都相当于一次IO操作，因此**红黑树的I/O操作会比B树多的多**。

 

hash 索引：如果只查询单个值的话，hash 索引的效率非常高。但是 **hash 索引有几个问题**：1）不支持范围查询；2）不支持索引值的排序操作；3）不支持联合索引的最左匹配规则。

 

B树索引：B树索相比于B+树，在进行范围查询时，需要做局部的中序遍历，可能要跨层访问，跨层访问代表着要进行额外的磁盘I/O操作；另外，B树的非叶子节点存放了数据记录的地址，会导致存放的节点更少，树的层数变高。

### 8.MySQL 中的索引叶子节点存放的是什么？

**MyISAM和InnoDB都是采用的B+树作为索引结构，但是叶子节点的存储上有些不同。**

**MyISAM**：**主键索引和辅助索引（普通索引）的叶子节点都是存放 key 和 key 对应数据行的地址**。在MyISAM 中，主键索引和辅助索引没有任何区别。

**InnoDB**：**主键索引存放的是 key 和 key 对应的数据行。辅助索引存放的是 key 和 key 对应的主键值**。因此在使用辅助索引时，通常需要检索两次索引，首先检索辅助索引获得主键值，然后用主键值到主键索引中检索获得记录。



### 9.什么是聚簇索引（聚集索引）？

聚簇索引并不是一种单独的索引类型，而是一种**数据存储方式**。**聚簇索引将索引和数据行放到了一块**，找到索引也就找到了数据。因为无需进行回表操作，所以效率很高。

**InnoDB 中必然会有，且只会有一个聚簇索引**。通常是主键，如果没有主键，则优先选择非空的唯一索引，如果唯一索引也没有，则会创建一个隐藏的row_id 作为聚簇索引。至于为啥会只有一个聚簇索引，其实很简单，**因为我们的数据只会存储一份**。

**而非聚簇索引则将数据存储和索引分开，找到索引后，需要通过对应的地址找到对应的数据行。MyISAM 的索引方式就是非聚簇索引。**

### 10.什么是回表查询？

InnoDB 中，对于主键索引，只需要走一遍主键索引的查询就能在叶子节点拿到数据。

而**对于普通索引，叶子节点存储的是 key + 主键值，因此需要再走一次主键索引，通过主键索引找到行记录，这就是所谓的回表查询**，先定位主键值，再定位行记录。

### 11.走普通索引，一定会出现回表查询吗？

**不一定，如果查询语句所要求的字段全部命中了索引，那么就不必再进行回表查询。**

很容易理解，有一个 user 表，主键为 id，name 为普通索引，则再执行：select id, name from user where name = 'joonwhee' 时，通过name 的索引就能拿到 id 和 name了，因此无需再回表去查数据行了。

### 12.那你知道什么是覆盖索引（索引覆盖）吗？

覆盖索引是 SQL-Server 中的一种说法，上面讲的例子其实就实现了覆盖索引。具体的：**当索引上包含了查询语句中的所有列时，我们无需进行回表查询就能拿到所有的请求数据，因此速度会很快。**

### 13.联合索引（复合索引）的底层实现？最佳左前缀原则？？？

**联合索引底层还是使用B+树索引，并且还是只有一棵树**，只是此时的排序会：首先按照第一个索引排序，在第一个索引相同的情况下，再按第二个索引排序，依次类推。

这也是为什么有“**最佳左前缀原则**”的原因，因为右边（后面）的索引都是在左边（前面）的索引排序的基础上进行排序的，如果没有左边的索引，单独看右边的索引，其实是无序的。

还是以字典为例，我们如果要查第2个字母为 k 的，通过目录是无法快速找的，因为首字母 A - Z 里面都可能包含第2个字母为 k 的。

### 14.union 和 union all 的区别

union all：**对两个结果集直接进行并集操作，记录可能有重复，不会进行排序**。

union：**对两个结果集进行并集操作，会进行去重，记录不会重复，按字段的默认规则排序**。

因此，从效率上说，UNION ALL 要比 UNION 更快。

### 15.B+树中一个节点到底多大合适？

**页或页的倍数最为合适**。因为如果一个节点的大小小于1页，那么读取这个节点的时候其实也会读出1页，造成资源的浪费。所以为了不造成浪费，所以最后把一个节点的大小控制在1页、2页、3页等倍数页大小最为合适。

这里说的“页”是 MySQL 自定义的单位（和操作系统类似），**MySQL 的 Innodb 引擎中1页的默认大小是16k**，可以使用命令SHOW GLOBAL STATUS LIKE 'Innodb_page_size' 查看。

### 16.为什么一个节点为1页就够了？

Innodb中，B+树中的一个节点存储的内容是：

**非叶子节点：key + 指针**

**叶子节点：数据行（key 通常是数据的主键）**

对于叶子节点：我们假设1行数据大小为1k（对于普通业务绝对够了），那么1页能存16条数据。

对于非叶子节点：key 使用 bigint 则为8字节，指针在 MySQL 中为6字节，一共是14字节，则16k能存放 16 * 1024 / 14 = 1170个。那么一颗高度为3的B+树能存储的数据为：1170 * 1170 * 16 = 21902400（千万级）。

所以在 **InnoDB 中B+树高度一般为3层时，就能满足千万级的数据存储**。在查找数据时一次页的查找代表一次IO，所以通过主键索引查询通常只需要1-3次 IO 操作即可查找到数据。千万级别对于一般的业务来说已经足够了，所以一个节点为1页，也就是16k是比较合理的。

### 17.什么是 Buffer Pool？

**Buffer Pool 是 InnoDB 维护的一个缓存区域**，**用来缓存数据和索引在内存**中，主要用来**加速数据的读写**，如果 Buffer Pool 越大，那么 MySQL 就越像一个内存数据库，**默认大小为 128M**。

InnoDB 会将那些热点数据和一些 InnoDB 认为即将访问到的数据存在 Buffer Pool 中，以提升数据的读取性能。

InnoDB 在修改数据时，如果数据的页在 Buffer Pool 中，则会直接修改 Buffer Pool，此时我们称这个页为**脏页**，InnoDB 会以一定的频率将脏页刷新到磁盘，这样可以尽量减少磁盘I/O，提升性能。

mysql8.0去除缓存。

### 18.InnoDB 四大特性知道吗？？？？？？

**插入缓冲（insert buffer）：**

索引是存储在磁盘上的，所以对于索引的操作需要涉及磁盘操作。如果我们使用自增主键，那么在插入主键索引（聚簇索引）时，只需不断追加即可，**不需要磁盘的随机 I/O**。但是如果我们使用的是普通索引，大概率是无序的，此时就涉及到磁盘的随机 I/O，而随机I/O的性能是比较差的（Kafka 官方数据：**磁盘顺序I/O的性能是磁盘随机I/O的4000~5000倍**）。

 

因此，InnoDB 存储引擎设计了 Insert Buffer ，对于非聚集索引的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池（Buffer pool）中，若在，则直接插入；若不在，则先放入到一个 Insert Buffer 对象中，然后再以一定的频率和情况进行 Insert Buffer 和辅助索引页子节点的 merge（合并）操作，这时通常能将多个插入合并到一个操作中（因为在一个索引页中），这就大大提高了对于非聚集索引插入的性能。

插入缓冲的使用需要满足以下两个条件：1）索引是辅助索引；2）索引不是唯一的。

 

因为在插入缓冲时，数据库不会去查找索引页来判断插入的记录的唯一性。如果去查找肯定又会有随机读取的情况发生，从而导致 Insert Buffer 失去了意义。

 

**二次写（double write）：**

脏页刷盘风险：InnoDB 的 page size一般是16KB，操作系统写文件是以4KB作为单位，那么每写一个 InnoDB 的 page 到磁盘上，操作系统需要写4个块。于是可能出现16K的数据，写入4K 时，发生了系统断电或系统崩溃，只有一部分写是成功的，这就是 partial page write（部分页写入）问题。这时会出现数据不完整的问题。

这时是无法通过 redo log 恢复的，因为 redo log 记录的是对页的物理修改，如果页本身已经损坏，重做日志也无能为力。

 

doublewrite 就是用来解决该问题的。doublewrite 由两部分组成，一部分为内存中的 doublewrite buffer，其大小为2MB，另一部分是磁盘上共享表空间中连续的128个页，即2个区(extent)，大小也是2M。

为了解决 partial page write 问题，当 MySQL 将脏数据刷新到磁盘的时候，会进行以下操作：

1）先将脏数据复制到内存中的 doublewrite buffer

2）之后通过 doublewrite buffer 再分2次，每次1MB写入到共享表空间的磁盘上（顺序写，性能很高）

3）完成第二步之后，马上调用 fsync 函数，将doublewrite buffer中的脏页数据写入实际的各个表空间文件（离散写）。

 

如果操作系统在将页写入磁盘的过程中发生崩溃，InnoDB 再次启动后，发现了一个 page 数据已经损坏，InnoDB 存储引擎可以从共享表空间的 doublewrite 中找到该页的一个最近的副本，用于进行数据恢复了。

 

**自适应哈希索引(adaptive hash index）：**

哈希（hash）是一种非常快的查找方法，一般情况下查找的时间复杂度为 O(1)。但是由于不支持范围查询等条件的限制，InnoDB 并没有采用 hash 索引，但是如果能在一些特殊场景下使用 hash 索引，则可能是一个不错的补充，而 InnoDB 正是这么做的。

 

具体的，InnoDB 会监控对表上索引的查找，如果观察到某些索引被频繁访问，索引成为热数据，建立哈希索引可以带来速度的提升，则建立哈希索引，所以称之为自适应（adaptive）的。自适应哈希索引通过缓冲池的 B+ 树构造而来，因此建立的速度很快。而且不需要将整个表都建哈希索引，InnoDB 会自动根据访问的频率和模式来为某些页建立哈希索引。

 

**预读（read ahead）：**

InnoDB 在 I/O 的优化上有个比较重要的特性为预读，当 InnoDB 预计某些 page 可能很快就会需要用到时，它会异步地将这些 page 提前读取到缓冲池（buffer pool）中，这其实有点像空间局部性的概念。

 

空间局部性（spatial locality）：如果一个数据项被访问，那么与他地址相邻的数据项也可能很快被访问。

 

InnoDB使用两种预读算法来提高I/O性能：线性预读（linear read-ahead）和随机预读（randomread-ahead）。

 

其中，线性预读以 extent（块，1个 extent 等于64个 page）为单位，而随机预读放到以 extent 中的 page 为单位。线性预读着眼于将下一个extent 提前读取到 buffer pool 中，而随机预读着眼于将当前 extent 中的剩余的 page 提前读取到 buffer pool 中。

 

线性预读（Linear read-ahead）：线性预读方式有一个很重要的变量 innodb_read_ahead_threshold，可以控制 Innodb 执行预读操作的触发阈值。如果一个 extent 中的被顺序读取的 page 超过或者等于该参数变量时，Innodb将会异步的将下一个 extent 读取到 buffer pool中，innodb_read_ahead_threshold 可以设置为0-64（一个 extend 上限就是64页）的任何值，默认值为56，值越高，访问模式检查越严格。

 

随机预读（Random read-ahead）: 随机预读方式则是表示当同一个 extent 中的一些 page 在 buffer pool 中发现时，Innodb 会将该 extent 中的剩余 page 一并读到 buffer pool中，由于随机预读方式给 Innodb code 带来了一些不必要的复杂性，同时在性能也存在不稳定性，在5.5中已经将这种预读方式废弃。要启用此功能，请将配置变量设置 innodb_random_read_ahead 为ON。



### 19.说说共享锁和排他锁？

共享锁又称为读锁，简称**S锁**，顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是**只能读不能修改**。

 

排他锁又称为写锁，简称**X锁**，顾名思义，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务可以对数据就行读取和修改。

 

常见的几种 SQL 语句的加锁情况如下：

select * from table：不加锁

**update/insert/delete：排他锁**

**select * from table where id = 1 for update：id为索引，加排他锁**

**select * from table where id = 1 lock in share mode：id为索引，加共享锁**

### 20.InnoDB 的行锁是怎么实现的？

InnoDB 行锁是**通过索引上的索引项来实现**的。意味者：只有通过索引条件检索数据，InnoDB 才会使用行级锁，否则，InnoDB将使用表锁！

 

对于主键索引：直接锁住锁住主键索引即可。

对于普通索引：先锁住普通索引，接着锁住主键索引，这是因为一张表的索引可能存在多个，通过主键索引才能确保锁是唯一的，不然如果同时有2个事务对同1条数据的不同索引分别加锁，那就可能存在2个事务同时操作一条数据了。

### 21.InnoDB 锁的算法有哪几种？

Record lock：记录锁，单条索引记录上加锁，锁住的永远是索引，而非记录本身。 (select name from user_table where id = 1 for update)

Gap lock：**间隙锁**，在索引记录之间的间隙中加锁，或者是在某一条索引记录之前或者之后加锁，并不包括该索引记录本身。(select name from user_table where id < 2 for update)

Next-key lock：Record lock 和 Gap lock 的结合**，即除了锁住记录本身，也锁住索引之间的间隙。 (select name from user_table where id < =2 for update)**



### 22.MySQL 如何实现悲观锁和乐观锁？

乐观锁：更新时带上版本号（cas更新）

**悲观锁：使用共享锁和排它锁，select...lock in share mode，select…for update**。

### 23.InnoDB 和 MyISAM 的区别？

![](..\截图\MYISAM_Innodb.png)

### 24.存储引擎的选择？

没有特殊情况，使用 InnoDB 即可。如果表中绝大多数都只是读查询，可以考虑 MyISAM。

### 25.explain 用过吗，有哪些字段分别是啥意思？



### 26.如何做慢 SQL 优化？

首先要搞明白慢的原因是什么：是查询条件没有命中索引？还是 load 了不需要的数据列？还是数据量太大？所以优化也是针对这三个方向来的。

1. 首先用 explain 分析语句的执行计划，查看使用索引的情况，是不是查询没走索引，如果可以加索引解决，优先采用加索引解决。
2. 分析语句，**看看是否存在一些导致索引失效的用法**，是否 load 了额外的数据，是否加载了许多结果中并不需要的列，对语句进行分析以及重写。
3. **如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行垂直拆分或者水平拆分。**

### 27.说说 MySQL 的主从复制？

MySQL主从复制**涉及到三个线程**，一个运行在主节点（Log Dump Thread），其余两个（I/O Thread，SQL Thread）运行在从节点，如下图所示

![](..\截图\Mysql主从复制.png)

主从复制默认是异步的模式，具体过程如下。

1）从节点上的I/O 线程连接主节点，并请求从指定日志文件（bin log file）的指定位置（bin log position，或者从最开始的日志）之后的日志内容；

2）主节点接收到来自从节点的 I/O请求后，读取指定文件的指定位置之后的日志信息，返回给从节点。返回信息中除了日志所包含的信息之外，还包括本次返回的信息的 **bin-log file 以及 bin-log position**；从节点的 I/O 进程接收到内容后，将接收到的日志内容更新到 relay log 中，并将读取到的 bin log file（文件名）和position（位置）保存到 master-info 文件中，以便在下一次读取的时候能够清楚的告诉 Master “我需要从某个bin-log 的哪个位置开始往后的日志内容”；

3）从节点的 SQL 线程检测到 relay-log 中新增加了内容后，会解析 relay-log 的内容，并在本数据库中执行。



### 28.异步复制，主库宕机后，数据可能丢失？

半同步复制：

修改语句写入bin log后，不会立即给客户端返回结果。而是首先通过log dump 线程将 binlog 发送给从节点，从节点的 I/O 线程收到 binlog 后，写入到 relay log，然后返回 ACK 给主节点，主节点 收到 ACK 后，再返回给客户端成功。



同步复制的特点：

确保事务提交后 binlog 至少传输到一个从库

不保证从库应用完这个事务的 binlog

性能有一定的降低，响应时间会更长

网络异常或从库宕机，卡主主库，直到超时或从库恢复

 

全同步复制：主节点和所有从节点全部执行了该事务并确认才会向客户端返回成功。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

### 29.主库写压力大，从库复制很可能出现延迟？

可以使用**并行复制**（并行是指从库多个SQL线程并行执行 relay log），**解决从库复制延迟的问题**。

MySQL 5.7 中引入**基于组提交的并行复制**，其核心思想：**一个组提交的事务都是可以并行回放，因为这些事务都已进入到事务的 prepare 阶段，则说明事务之间没有任何冲突（否则就不可能提交）。**

判断事务是否处于一个组是通过 last_committed 变量，last_committed 表示事务提交的时候，上次事务提交的编号，如果事务具有相同的 last_committed，则表示这些事务都在一组内，可以进行并行的回放。







