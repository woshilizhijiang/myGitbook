# 设计模式应用

### Decorator Pattern（装饰模式）

### 学习的契机：

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



























