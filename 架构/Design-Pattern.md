# 设计模式应用

### Decorator Pattern（装饰模式）

#### 学习的契机：

SpringBoot分析spring5的webFlux时；HttpHandler、WebHandler、WebHandlerDecorator、HttpWebHandlerAdapter

#### 定义：

#### 组成：

- Component：接口或者抽象类；核心对象，也是原始对象
- Concretecomponent：实现类，装饰类核心
- Decorator：装饰角色；接口或者抽象类
- 具体装饰角色：

#### 优点：

Decorator类和基类可独立发展，不互相耦合。Decorator从外部修饰基类。

Decorator Pattern是继承关系的一个替换方案。实现is-a

动态地扩展一个实现类功能。

#### 缺点：

设计复杂：多层装饰比较复杂。

#### 场景

- 扩展一个类的功能，或给一个增加附加功能；
- 动态给一个对象增加功能，该部分功能可动态取消；
- 为一些兄弟类进行改装或者加装功能，首选Decorator Pattern。



### Proxy pattern(代理模式)

#### 学习的契机：

spring-AOP分析时，遇到静态代理和动态代理问题。

#### 定义：

为其他对象提供一种代理以控制对这个对象的访问

#### 组成：

- Subject：抽象主题角色
- RealSubjet：具体主题角色
- Proxy：代理主题角色

#### 优点：

- 职责清晰
- 高扩展性
- 智能化

#### 缺点：

静态代理缺点：
代理方法多，代理代码过分设计编写；

#### 分类

- 普通代理：**代理类代理一切**
- 强制代理：**实际类制定代理类**

```java
//
//普通代理
IGamePlayer proxy = new GamePlayerProxy("李元昕");
proxy.login("liyuanxin","password");
proxy.killBoss();
proxy.upgrade();

System.out.println("************************************************");

//强制代理
IGamePlayer player = new GamePlayer("lisi");
//        IGamePlayer proxy2 =  new GamePlayerProxy(player); //不成
IGamePlayer proxy2 = player.getProxy();
proxy2.login("lisi","password1");
proxy2.killBoss();
proxy2.upgrade();
```

- 动态代理:
  核心：
  **JDK**
  java .lang.reflect.InvocationHandler是基类 invoke(Object proxy,Method method, Object[] args)方法

  java.lang.ClassLoader:
  java.lang.reflect.Proxy:

  ```java
  IGamePlayer player = new GamePlayer("李红静");
  InvocationHandler handler = new GamePlayIH(player);
  
  ClassLoader cl = player.getClass().getClassLoader();
  IGamePlayer proxy = (IGamePlayer)Proxy.newProxyInstance(cl,new Class[]{IGamePlayer.class},handler);
  proxy.getProxy();
  proxy.login("lihongjing","1qaz!QAZ");
  proxy.killBoss();
  proxy.upgrade();
  ```

#### 场景

**Spring-AOP动态代理和静态代理**







### Singleton pattern单例模式

#### 学习的契机：

​		枚举单例，java反射报错无法实例化。 引申对单例模式再度深入学习。
​	问题：

```java
package com.pattern.singleton.enumType;
public enum Enumsingleton {
    INSTANCE;
    public Enumsingleton getInstance(){
        return INSTANCE;
    }
}

package com.pattern.singleton;

import com.pattern.singleton.enumType.Enumsingleton;

import java.lang.reflect.Constructor;

public class SingletonClient {

    public static void main(String[] args) throws Exception{
        Constructor<Enumsingleton> constructor1 = Enumsingleton.class.getDeclaredConstructor();
        constructor1.setAccessible(true);
        Enumsingleton e3 = constructor1.newInstance();

        /**
         * Exception in thread "main" java.lang.NoSuchMethodException: com.pattern.singleton.enumType.Enumsingleton.<init>()
         * 	at java.lang.Class.getConstructor0(Class.java:3082)
         * 	at java.lang.Class.getDeclaredConstructor(Class.java:2178)
         * 	at com.pattern.singleton.Singleton.main(Singleton.java:34)
         */
        System.out.println(e3);
    }
}



   /*错误原因java.lang.reflect.Constructor*/
    @CallerSensitive
    public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        //枚举报错原因开始
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
	
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }



```

```反射异常
反射构造器调用getDeclaredConstructor()方法时，枚举生成如下：
private com.pattern.singleton.enumType.Enumsingleton(java.lang.String,int)


```



#### 定义：

​		确保一个类有且只有一个实例，自行实例化并向整个系统提供这个实例。

#### 组成：

​		单例类：
​		懒汉式单例：（第一次）使用加载或者叫运行时加载。

​					枚举类型单例：java反射无法创建实例

​		饿汉式单例：系统初始化时加载。

​		

#### 优点：

1. 减少内存开销，避免频繁创建、销毁对象的消耗。
2. 减少系统性能开销，

#### 缺点：

1. 没有接口类，不易扩展
2. 对测试不利
3. 与单一性原则相违背

#### 场景：







### 责任链模式-chain of responsibility pattern

#### 学习的契机：

osp使用apache commons chain进行各个组件功能开发，分析框架时使用；netty的Channel是否有相似之处

#### 定义：

使多个对象都有机会处理请求，避免请求的发送者和接受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止。

#### 组成：

抽象处理者：
具体处理者：
客户端类：

#### 优点：



#### 缺点：



#### 场景：



### 命令模式-commond pattern

#### 学习的契机：

osp使用apache commons chain进行各个组件功能开发，分析框架时使用。

#### 定义：

将一个请求封装成一个对象，从而让你使用不同的请求把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销和恢复功能。

#### 组成：

receive接收者角色：干活角色，命令传到该处被执行的。

commond命令角色：申明所有需要执行的命令。

invoker调用者角色：接收并执行命令。

#### 优点：

类之间解耦：

可扩展性：

命令模式结合其他模式更优秀：

#### 缺点：

command子类数量繁多。

#### 场景：



### 模板方法

**概念层次**
 是某个抽象对象，具有不同的具体对象，需要完成一件事情
 （1）**有相同的步骤，但单个步骤的实现可能不同**
 （2）所以完成一件事情这个行为，**控制步骤顺序**
 （3）单个步骤行为可以抽象，让具体对象去实现

父类定义一个方法，这个方法里面调用一些抽象方法，这些抽象方法让子类去实现

父类调用抽象方法的方法（在父类内部指定步骤（1）,（2）），就是模板方法



### 外观模式

##### 定义：

facade 是一种通过为多个复杂的子系统提供一致的接口。是”迪米特法则“的典型应用。

##### 优点：

1.降低子系统与客户端之间的耦合度，使子系统变化不会影响用它的客户类。

2.对客户屏蔽了子系统，是客户”降本增效“。

3.降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程。

##### 缺点：

1.不能限制客户使用子类系统。

2.增加新的子类可能需要修改外观类或客户端的源代码，违背了“开闭原则”。

##### 结构组成：

外观角色（facade）：聚合多个子系统，提供一个共同的接口。

子系统角色（sub system）：实现系统各个部分功能。

客户角色（client）：访问者

### 观察者模式

对象的行为模式

##### 定义：

指多个对象间存在一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。这种模式有时又称作发布-订阅模式、模型-视图模式，它是对象行为型模式。

##### 优点：

1. 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。
2. 目标与观察者之间建立了一套触发机制。

##### 缺点：

1. 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
2. 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

##### 结构组成：

抽象主题：

具体主题：

抽象观察者：

具体观察者：















