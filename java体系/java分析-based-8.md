# 1.jdk8经典分析

## 基础

### 枚举详解（enum:java.lang.Enum）

#### 声明枚举类

```java
[public] enum 枚举类型名称
  [implements 接口]
{
  枚举值；
  变量成员声明及初始化;
  方法声明及方法体;
}
```

枚举值：即对象实例
变量和常量

#### 特点

- 一个类
- 所有枚举类型都隐含继承（扩展）自java.lang.Enum，枚举类不能再继承其他任何类型；
- 包含方法和变量名称
- 构造方法必须是包内私有或者私有的，定义在枚举类开头的常量会自动创建，不能显示的调用枚举类的构造方法

#### 默认方法

- 静态values：用于获得枚举类型的枚举值的数组；
- toString：返回枚举值字符串描述
- valueOf：将字符串形式表示的枚举值转化为枚举类型对象
- ordinal：获得对象在枚举类型中的位置索引

#### 使用场景

常量、switch、向枚举中添加新的方法、覆盖枚举的方法、实现接口、使用接口组织枚举、



### 注解

#### annotation basic(基础注解)



#### meta-annotaions(元注解)

**1.java.lang.annotation.Retention**

```java
package java.lang.annotation;
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```



## lambda表达式

```用法
* 用lambda表达式实现Runnable
* lambda表达式替换匿名类
*
* lambda语法：
* (params) -> expression
* (params) -> statement
* (params) -> {statements}
```

### 5种类用法

- lambda使用foreach

  ```java
  List<String> features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");	
  //方法一
  features.forEach((n)->System.out.print.out(n));
      //方法二
      // 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
      // 看起来像C++的作用域解析运算符
      features.forEach(System.out::println);
  //        for (String feature : features) {
  //            System.out.println(feature);
  //        }
  ```
  
- lambda实现Runnable

  ```java
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println(Thread.currentThread().getName() + " : Before Java8, too much code for too little to do");
              }
          },"Thread1").start();
  
  
  
  new Thread(()->System.out.print.out(Thread.currentThread().getName() + " : In Java8, Lamda expression rocks!!"),
                  "Thread2")).start();
  ```

- lambda表达式和 函数式编程接口Predicate  :jdk8 java.util.funtion

```java
List<String> languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
System.out.println("Languages which starts with J :");
filter(languages, (str) -> ((String)str).startsWith("J") );

System.out.println("Print all languages :");
filter(languages, (str)->true);

System.out.println("Print null :");
filter(languages, (str)->false);

System.out.println("Print language whose length greater than 4:");
filter(languages, (str)->((String)str).length() > 4);

public static void filter(List<String> names, Predicate condition){
        names.forEach((name) -> {
            if (condition.test(name)){
                System.out.println(name + " ");
            }
        });
}
```

- lambda对象比较

```java
private void lambdaComparator(){
        Comparator<Apple> byWeight = new Comparator<Apple>() {
            @Override
            public int compare(Apple a1, Apple a2) {

                return a1.getWeight().compareTo(a2.getWeight());
            }
        };

        //Lambda
        Comparator<Apple> byWeightLambda = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

        System.out.println(byWeight);

    }
```



### lambda进化2.0

#### 两种方式对比

- 匿名内部类
- lambda函数编程
  **@FunctionalInterface作用就是标识一个接口为函数式接口**

#### lambda语法

多参数

   （1）. lambda表达式的基本格式为(x1,x2)->{表达式...};

   （2）. 在上式中，lambda表达式带有两个参数，此时参数类型可以省略，但两边的括号不能省略

   （3）. 如果表达式只有一行，那么表达式两边的花括号可以省略

无参数
	   ( )->{....表达式....}

   （1）.参数的括号不能省略，

   （2）.其他语法同多参数

```java
new Thread(() -> {
  //Runnable的run方法操作内容
  
}).start();
```

一个参数

   （1）.可以省略参数的括号和类型

   （2）.其他语法同多参数

#### jdk提供的函数式接口

包集中在：java.util.function

***Function：提供任意一种类型的参数，返回另外一个任意类型返回值。 R apply(T t);***

***Consumer：提供任意一种类型的参数，返回空值。 void accept(T t);***

***Supplier：参数为空，得到任意一种类型的返回值。T get();***

***Predicate：提供任意一种类型的参数，返回boolean返回值。boolean test(T t);***



#### 方法引用：

三种方式

- 类::实例方法
- 类::静态方法
- 对象::实例方法

```java
System.out::println    ==   (x) -> System.out.println(x);  //对象::实例方法
Math::pow              ==   (x,y) -> Math.pow(x,y) //类::静态方法
```

#### 自由变量的作用范围

```java
/**
com.java8.lambda.base.constructor.ConstructorDemo@1789e283
lambda express run...
com.java8.lambda.base.constructor.ConstructorDemo$1@4cee664c
anony function run...
说明
lambda的this是指包含lambda表达式的类
匿名内部类的this；表示匿名内部类本身
*/
public static void main(String[] args) {
        new ConstructorDemo().doWork1();
        new ConstructorDemo().doWork2();
    }

    public void doWork1(){
        Runnable runnable = ()->{
            System.out.println(this.toString());
            System.out.println("lambda express run...");
        };
        new Thread(runnable).start();
    }

    public void doWork2(){
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(this.toString());
                System.out.println("anony function run...");
            }
        };
        new Thread(runnable).start();
    }
```



#### 接口的静态方法和默认方法

以java.util.Comparator为示例

接口中引入默认方法设计到一个问题

（1）接口中的默认方法和父类中方法的冲突问题

（2）接口之间引用的冲突问题

对于第一个冲突，**java8规定类中的方法优先级要高于接口中的默认方法**，所以接口中默认方法复写Object类中的方法是没有意义的，因为所有的接口都默认继承自Object类使得默认方法一定会被覆盖。

对于第二个冲突，**java8强制要求子类必须复写接口中冲突的方法**



### Stream分析

位置java.util.stream.Stream

#### Stream特性

**1.stream不存储数据**

**2.stream不改变源数据**

**3.stream的延迟执行特性**



**5.原始类型流**

在数据量比较大的情况下，将基本数据类型（int,double...）包装成相应对象流的做法是低效的，因此，我们也可以直接将数据初始化为原始类型流，在原始类型流上的操作与对象流类似，我们只需要记住两点

1.原始类型流的初始化

2.原始类型流与流对象的转换

**6.并行流**

可以将普通顺序执行的流转变为并行流，只需要调用顺序流的parallel() 方法即可，如Stream.iterate(1, x -> x + 1).limit(10).parallel()。

**1） 并行流的执行顺序**

我们调用peek方法来瞧瞧并行流和串行流的执行顺序，peek方法顾名思义，就是偷窥流内的数据，peek方法声明为Stream<T> peek(Consumer<? super T> action);加入打印程序可以观察到通过流内数据





## java源码

### 集合类

#### 1.HashMap

底层实现由“数组+链表”改为“数组+链表+红黑树”，架构图如下，当链表节点较少仍是**“数组+链表”**，当链表节点大于8转为**“数组+红黑树”**。(改图不准确，红黑树节点无数据必须以nil来填充)

![](..\截图\HashMap架构图.png)

```java
//0.hash
//(n - 1) & hash = x % 2^n
//x mod 2^n = x & (2^n - 1)
//1.get方法
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //1.对table进行校验：table不为空 && table长度大于0 && 
    // table索引位置(使用table.length - 1和hash值进行位与运算)的节点不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 2.检查first节点的hash值和key是否和入参的一样，如果一样则first即为目标节点，直接返回first节点
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 3.如果first不是目标节点，并且first的next节点不为空则继续遍历
        if ((e = first.next) != null) {
            // 4.如果是红黑树节点，则调用红黑树的查找目标节点方法getTreeNode
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 5.执行链表节点的查找，向下遍历链表, 直至找到节点的key和入参的key相等时,返回该节点
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 6.找不到符合的返回空
    return null;
}

//2.put方法
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 1.校验table是否为空或者length等于0，如果是则调用resize方法进行初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 2.通过hash值计算索引位置，将该索引位置的头节点赋值给p，如果p为空则直接在该索引位置新增一个节点即可
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // table表该索引位置不为空，则进行查找
        Node<K,V> e; K k;
        // 3.判断p节点的key和hash值是否跟传入的相等，如果相等, 则p节点即为要查找的目标节点，将p节点赋值给e节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 4.判断p节点是否为TreeNode, 如果是则调用红黑树的putTreeVal方法查找目标节点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 5.走到这代表p节点为普通链表节点，则调用普通的链表方法进行查找，使用binCount统计链表的节点数
            for (int binCount = 0; ; ++binCount) {
                // 6.如果p的next节点为空时，则代表找不到目标节点，则新增一个节点并插入链表尾部
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 7.校验节点数是否超过8个，如果超过则调用treeifyBin方法将链表节点转为红黑树节点，
                    // 减一是因为循环是从p节点的下一个节点开始的
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 8.如果e节点存在hash值和key值都与传入的相同，则e节点即为目标节点，跳出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
         // 9.如果e节点不为空，则代表目标节点存在，使用传入的value覆盖该节点的value，并返回oldValu
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 10.如果插入节点后节点数超过阈值，则调用resize方法进行扩
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

//remove
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

##### 1.死循环问题

resize导致的节点顺序错乱

##### 2.Hashmap

1. **为什么要改成“数组+链表+红黑树”？**
   主要是为了提升**在 hash 冲突严重时（链表过长）的查找性能**，使用链表的查找性能是 O(n)，而使用红黑树是 O(logn)。
2. **那在什么时候用链表？什么时候用红黑树？**
   对于插入，**默认情况下是使用链表节点**。当同一个索引位置的节点在新增后达到9个（阈值8）：如果此时数组长度大于等于 64，则会触发链表节点转红黑树节点（treeifyBin）；而如果数组长度小于64，则不会触发链表转红黑树，而是会进行扩容，因为此时的数据量还比较小。
   对于移除，当同一个索引位置的节点在移除后达到 6 个，并且该索引位置的节点为红黑树节点，会触发红黑树节点转链表节点（untreeify）。
3. **为什么链表转红黑树的阈值是8？**
   我们平时在进行方案设计时，必须考虑的两个很重要的因素是：**时间和空间**。对于 HashMap 也是同样的道理，简单来说，**阈值为8是在时间和空间上权衡的结果**（这 B 我装定了）。
   **红黑树节点大小约为链表节点的2倍**，在节点太少时，红黑树的查找性能优势并不明显，付出2倍空间的代价作者觉得不值得。
   理想情况下，使用**随机的哈希码**，节点分布在 hash 桶中的频率遵循泊松分布，**按照泊松分布的公式计算，链表中节点个数为8时的概率为 0.00000006**（跟大乐透一等奖差不多，中大乐透？不存在的），这个概率足够低了，并且到8个节点时，红黑树的性能优势也会开始展现出来，因此8是一个较合理的数字。
4. **那为什么转回链表节点是用的6而不是复用8？**
   如果我们设置节点多于8个转红黑树，少于8个就马上转链表，**当节点个数在8徘徊时，就会频繁进行红黑树和链表的转换，造成性能的损耗**。
5. **那 HashMap 有哪些重要属性？分别用于做什么的？**
   1）size：HashMap 已经存储的节点个数；2）threshold：扩容阈值，当 HashMap 的个数达到该值，触发扩容。3）loadFactor：负载因子，扩容阈值 = 容量 * 负载因子。
6. **threshold 除了用于存放扩容阈值还有其他作用吗？**
   默认初始容量是16。HashMap 的容量必须是2的N次方，HashMap 会根据我们传入的容量计算一个大于等于该容量的最小的2的N次方，例如传 9，容量为16。
7. **你说 HashMap 的容量必须是 2 的 N 次方，这是为什么？**
   计算索引位置的公式为：(n - 1) & hash，当 n 为 2 的 N 次方时，n - 1 为低位全是 1 的值，此时任何值跟 n - 1 进行 & 运算的结果为该值的低 N 位，达到了和取模同样的效果，实现了均匀分布。实际上，这个设计就是基于公式：x mod 2^n = x & (2^n - 1)，因为 & 运算比 mod 具有更高的效率。
8. **介绍一下死循环问题？**
   导致死循环的根本原因是 JDK 1.7 扩容采用的是“头插法”，会导致同一索引位置的节点在扩容后顺序反掉。而 JDK 1.8 之后采用的是“尾插法”，扩容后节点顺序不会反掉，不存在死循环问题。

#### 2.ConcurrentHashMap

jdk1.7采用分段；jdk1.8开始用“CAS+Synchronized”





### 多线程

#### 1.ThreadPoolExecutor线程池基类

java.util.concurrent.ThreadPoolExecutor



#### 2.AQS多线程基类

java.util.concurrent.locks.AbstractQueuedSynchronizer



hasQueuedPredecessors



#### 3.ReentrantLock锁

公平锁和非公平锁在 tryAcquire(int)是否有hasQueuedPredecessors()

```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }


public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```



### 4.关键字

#### 1.synchronized分析-衍生出的问题

非公平锁；

锁代码块和锁方法区别；

monitor机制；

锁优化演变历史：（jdk1.5-jdk1.6）

对象头信息理解





## 多线程

### 1.创建线程的两种方式

### 2.启动线程

### 3.停止线程

#### 线程停策略

优先选择：传递中断

不想或者无法传递：恢复中断

不应该屏蔽中断



#### Thread方法分析

线程内抛异常出问题的原因：
Thread.currentThread().isInterrupted()不生效原因是sleep抛出InterruptedException

if any thread has interrupted the current thread. **The  <i>interrupted status</i> of the current thread is  cleared when this exception is thrown**.

阻塞状态被清除

```java
/**
     * Causes the currently executing thread to sleep (temporarily cease
     * execution) for the specified number of milliseconds, subject to
     * the precision and accuracy of system timers and schedulers. The thread
     * does not lose ownership of any monitors.
     *
     * @param  millis
     *         the length of time to sleep in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static native void sleep(long millis) throws InterruptedException;
```

### 4.Thread及Object常用方法



**方法概览**

| 类               | 方法名                        | 简介                   |
| :--------------- | ----------------------------- | ---------------------- |
| java.lang.Thread | sleep相关                     |                        |
|                  | join                          | 等待其他线程执行完毕   |
|                  | yield相关                     | 放弃已经获取的CPU资源  |
|                  | currentThread                 | 获取当前线程的引用     |
|                  | start，run相关                | 启动线程相关           |
|                  | interrupt相关                 | 中断相关               |
|                  | **stop，suspend，resume相关** | **废弃方法**           |
| java.lang.Object | wait/notify/notifyAll相关     | 让现场暂时休息或者唤醒 |

主讲细则：

1. 方法概述
2. wait, notify,notifyAll方法详解
3. sleep方法详解
4. **join方法(main线程wait方法，Thread源码join(long millis))**
5. yield方法
6. currentThread引用
7. start和run
8. 过期方法

#### wait，notify，notifyAll方法详解

问题：**wait阻塞阶段 四种情况才能使线程被唤醒**

1. **另一个thread调用该对象的notify()方法且刚好被唤醒线程是本线程；**
2. **另一个thread调用该对象的notifyAll()方法；**
3. **过了wait(long timeout) 规定的超时时间，如果timeout为0则永久等待；**
4. **线程自生调用了interrupt()。**



常见面试题：

1. 用两个线程实现交替打印0～100的奇偶数（wait，notify；直接用synchronized）

2. **为什么wait()需要同步代码块内使用，而sleep()不需要？**

   主要是为了**防止死锁和永久等待的发生**，使通信变得可靠，如果不放在同步代码块里面很有可能在第一个线程wait()之前，cpu切换到另一个线程执行，而另一个线程里面执行了所有的notify()，然后又切回来执行wait()但是设计的逻辑是想执行完wait()之后再执行notify()去唤醒它，可是在没有synchronized保护之后就可以在执行wait()之前突然切换进第二个线程方法执行nortify()再切换回来，这样对方都已经把notify()都执行完毕了，这个导致刚进入wait()状态的线程永远没有人去唤醒它，导致死锁的发生。
   sleep本身是针对自己单独线程的，和其他关系并不大，所以并不需要放到同步代码块去做。

4. **为什么线程通信的方法wait(),notify(),notifyAll()被定义在Object类里？sleep定义在Thread中？**

   在java中的wait(),notify()和notifyaAll()是**锁级别的操作**，锁是属于某一个对象的，每一个对象的**对象头中都是含有几位用来保存锁的状态**的，所以这个锁实际上是绑定在某一对象中，而并不是线程中。同样道理我们反过来想假设定义这几个方法在线程中，就会造成很大的局限性。我们经常会遇到某一个线程持有多个锁，并且这些锁之间是相互配合的，如果定义在Thread类里面就没有办法去实现这些灵活的逻辑了。所以Java所提供的原始的锁对于每一个对象都是适用的，他就把这样的方法定义在object里面。
   
5. **wait方法是属于Object对象的，那调用Thread.wait()会怎样？**

   我们把它当做一个普通的锁是没有问题的，我们可以把一个类的实例当做一把锁去做它的wait()方法和notify,但是对于Thread这个类它非常特殊，就是**线程退出的时候它会自动地区执行notify()**，我们可以在**JVM的源码中去看到这部分**，这样会使我们设计的流程受到干扰，所以我们创建锁对象的时候不要用Thread类，它不适合作为锁对象，其他的类都比较适合，我们可以用普通的object类去实现就足够了。
   
6. 如何选择notify或者notifyAll？

   主要还是考虑到是要唤醒多个线程还是唤醒一个线程

   

7. **notifyAll之后所有线程都会再次抢夺锁，如果某个线程抢夺失败怎么办？**
   假设有五个线程都处于WAITING状态现在都唤醒了，同时去抢这把锁，但是只有一个线程能抢到这把锁。**没有抢到锁的线程还是回到WAITING状态等待锁释放再去抢或者说再去等待调度器的调度**，直到自己拿到这把锁去做下一步的操作。

   

8. 能不能用suspend()和resume()来阻塞线程可以吗？为什么？

   这些方法由于安全问题已经被弃用了，这个功能完全被wait()和notify()代替了



#### sleep方法总结

sleep方法可以让线程进入WAITING状态，并且不占用CPU资源，但是**不释放锁**，知道规定时间后再执行，休眠期间如果被中断，会抛出异常清除中断状态。

#### wait/notify，sleep异同？（方法属于哪个对象？线程状态怎么切换？）

相同：

1. 阻塞
2. 响应中断

不同：

1. wait/notify必须在同步方法中；
2. wait/notify释放锁，sleep不释放锁；
3. 指定时间
4. 所属类



### 5.多线程问题

#### 1.线程之间通信

线程之间通信有共享内存和消息传递，java才有共享内存。详见JMM

- Object类 wait、notify、notifyAll方法

- JUC中ReentrantLock：newCondition方法、Condition：await、signal、signalAll方法
  **ReentrantLock和管理多个Condition对象，形成多管道锁。**
  
  

#### 3.独占和共享锁

- ReentrantLock synchronized等独占锁
- CountdownLatch、CylicBarrier、Semephore属于共享锁

#### 2.ThreadPoolTaskExecutor

![](..\截图\ThreadPoolTaskExecutor执行流程.png)



- 五种状态

  ```java
      private static final int RUNNING    = -1 << COUNT_BITS;
      private static final int SHUTDOWN   =  0 << COUNT_BITS;
      private static final int STOP       =  1 << COUNT_BITS;
      private static final int TIDYING    =  2 << COUNT_BITS;
      private static final int TERMINATED =  3 << COUNT_BITS;
  ```

  ![](..\截图\线程池生命周期.png)



- 阻塞队列
  ![](..\截图\ThreadPoolExecutor阻塞队列.png)

## NIO







## GC回收机制

### ZGC

特点

- **停顿时间不超过10ms**
- **停顿时间不会随着堆增大而增加**
- 支持8MB~4TB级别堆（未来16TB）

#### GC之痛

介绍实际业务中遇到的 GC 痛点，并分析 CMS 收集器和 G1 收集器停顿时间瓶颈；  

### GC基础概念

- GC包含三个概念

Garbage Collection:垃圾收技术
Garbage Collector:垃圾收集器
Garbage Collecting:垃圾收集动作

- Mutator：**生成垃圾角色**
- TLAB（Thread local allocation buffer）
  **基于CAS独享线程可以优先分配Eden中一块内存。**
- Card Table
  卡表：

### JVM内存划分

![](..\截图\java8内存结构.png)



## java提供SPI接口类

### SPI定义

SPI，全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的API，它可以用于很多框架的扩展，常见的如Java JDBC、Spring、SpringBoot、Dubbo等。









# 2.effective java

### 1.枚举和注解



















































​	
