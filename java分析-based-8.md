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
  features.forEach((n)->System.out.print.out(n))
    
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

- JButton使用

```java
private void lambdaTrans(){
        JButton show = new JButton("Show");
//        show.addActionListener(new ActionListener() {
//            @Override
//            public void actionPerformed(ActionEvent e) {
//                System.out.println("Event handling without lambda expression is boring");
//            }
//        });

        show.addActionListener((e) -> {
            System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
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





## 多线程

### 1.创建线程的两种方式



### 2.启动线程



### 3.停止线程

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

