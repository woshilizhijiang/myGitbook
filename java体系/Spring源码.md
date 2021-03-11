## Spring源码

基于5.1.1.BUILD-SNAPSHOT

### IOC

####  一、IoC 之深入理解 Spring IoC

IOC(inversion of control)，或者叫DI（Dependency injection）依赖注入。

如何理解“控制反转”呢？关键在于我们需要回答如下四个问题：

1. 谁控制谁
2. 控制什么
3. 为何是反转
4. 哪些方面反转了

**所谓IoC，就是由Spring Ioc容器来负责对象的生命周期和对象之间的关系。**



![1](..\截图\IOC.png)

1. ##### 注入形式

   构造方法注入、setter方法注入、接口注入

   1. 构造器注入
   2. setter方法注入
   3. 接口方式注入

2. ##### 各个组件

   1. Resource体系
      org.springframework.core.io.Resource，对资源的抽象。它的每一个实现类都代表了一种资源的访问策略，如ClasspathResource、RLResource、FileSystemResource

   2. ResourceLoader体系
       org.springframework.core.io.ResourceLoader，资源加载，对资源进行统一加载。

   3. BeanFactory体系
      org.springframework.beans.factory.BeanFactory，纯粹bean容器，Ioc必备数据结构，其中BeanDefinition定义它结构。
      **BeanFactory 有三个直接子类 ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory；**
      **DefaultListableBeanFactory 为最终默认实现，它实现了所有接口**

   4.  BeanDefinition体系

      org.springframework.beans.factory.config.BeanDefinition，用来描述Spring中的Bean对象。

   5.  BeanDefinitionReader体系
      读取spring配置文件内容，并将其转换成IOC容器内部的数据结构：BeanDefinition

   6. ApplocationContext体系

      org.springframework.context.ApplicationContext，应用上下文，继承BeanFactory，是对其扩展升级，
      ![1](..\截图\ApplicationContext.png)

      继承 org.springframework.context.MessageSource 接口，提供国际化的标准访问策略。
      继承 org.springframework.context.ApplicationEventPublisher 接口，提供强大的事件机制。
      扩展 ResourceLoader ，可以用来加载多种 Resource ，可以灵活访问不同的资源。
      对 Web 应用的支持。

   7.  





