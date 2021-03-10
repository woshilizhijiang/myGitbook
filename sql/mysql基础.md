



## MYSQL基础



### 一、基础语句

```sql
--1.版本查询
select version(); --苏宁 5.7.19-17-log

```





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

1. 连接池
2. 管理服务和工具组件
3. sql接口组件
4. 查询分析器
5. 性能优化组件
6. 缓存
7. 插件式引擎
8. 物理文件

#### innodb简单叙述

​	InnoDB存储引擎支持事务，主要处理OLTP方面应用；特点：**行锁设计、支持外键、读取不产生锁**；

ibd文件

- 表中数据存储：采用聚簇索引的方式，引申出非聚簇索引； innodb两大索引(clustered index 、non-clustered index)

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

loop分两大部分操作：每秒钟和每10秒
master thread潜在问题：
对IO有限制，

关键特性：innodb存储引擎包含插入缓冲、两次写、自适应哈希索引（adaptive hash index）、预读；

**插入缓冲：**

针对插入性能提升

**两次写：**

数据可靠性提升
![image-20210210113543717](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210210113543717.png)

**自适应哈希索引**

查找性能提升，理论上时间复杂度为O(1)

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
  frm
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
- B+树定义:待补充......简短介绍,B+树为磁盘或者其他直接存取辅助设备设计的一种平衡查找树。
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
  - 共享锁（S Lock），允许事务读一行数据
  - 排他锁（X Lock），允许事务删除或更新一行数据
  - ![image-20210224211541356](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210224211541356.png)
    X、S锁都为行锁
  - 

#### 7章 事务

- 








### MyISAM简单叙述

不支持事务，表锁、全文索引；对于OLAP操作速度快

- 存储引擎由MYD和MYI组成，分别存放数据和索引



### 基础操作

show engines;

![image-20210208175607005](C:\Users\20013649\AppData\Roaming\Typora\typora-user-images\image-20210208175607005.png)

alter table mytest engine=mysiam as select * from dlv_vm_info

show engine  innodb status ;

show variables like 'innodb_version'

#### sql4种标准隔离

1. read uncommited：未提交可读取
2. read commited：提交可读取
3. reapatable：可重复度
4. serializable：可串行化

#### 基础语句

```sql

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

```markdown


常用的类型有： ALL、index、range、 ref、eq_ref、const、system、NULL（从左到右，性能从差到好）

ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

index: Full Index Scan，index与ALL区别为index类型只遍历索引树

range:只检索给定范围的行，使用一个索引来选择行

ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system

NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。
```



