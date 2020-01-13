# jdk8经典分析

## 基础源码

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

- **静态values：用于获得枚举类型的枚举值的数组；**
- toString：返回枚举值字符串描述
- **valueOf：将字符串形式表示的枚举值转化为枚举类型对象**
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







## 并发线程

### 1.ThreadPoolExecutor线程池基类

java.util.concurrent.ThreadPoolExecutor







### 2.AbstractQueuedSynchronizer多线程基类

java.util.concurrent.locks.AbstractQueuedSynchronizer



hasQueuedPredecessors



### 3.ReentrantLock锁

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

​	对象头信息理解




## JVM





## NIO





