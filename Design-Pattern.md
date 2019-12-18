# 设计模式应用

### Decorator Pattern（装饰模式）

#### 学习的契机：

SpringBoot分析spring5的webFlux时；HttpHandler、WebHandler、WebHandlerDecorator、HttpWebHandlerAdapter



<img src="/Users/lzj11/Documents/webFlux01.png" alt="webFlux01" style="zoom:33%;" />



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





























