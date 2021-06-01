## Spring源码-芋道

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



<img src="..\截图\IOC.png" alt="1" style="zoom: 50%;" />

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

      #### 二、IoC之Spring统一资源加载策略





### 程序加载分析

### 1.源码输入核心

- AbstractApplicationContext图谱
  ![](..\截图\ApplicationContext.png)

  Spring加载过程
  <img src="..\截图\Spring加载工程2.jpg"  />

  <img src="..\截图\Spring加载工程.jpg"  />

```java
ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("GenericParameterMatchingTests-context.xml");
counterAspect = (CounterAspect) ctx.getBean("counterAspect");
GenericInterface<String> testBean = (GenericInterface<String>) ctx.getBean("testBean");
testBean.save("tt");
```

- ClassPathXmlApplicationContext 执行了配置文件加载到Resource中的动作
- org.springframework.context.support.**AbstractApplicationContext**
      **refresh方法**    
  - **DefaultListableBeanFactory**通过反射

```java
/*
*返回静态指定的ApplicationListener列表。
*/
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

			// Prepare this context for refreshing.
            // 1.准备上下文以供刷新，包含启动日期、活动标志、属性源执行初始化
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 2.告诉子类刷新内部bean工厂，
            // 提供外部做扩展，相当于主流程过滤器
            // 判断有bean工厂 DefaultListableBeanFactory
            // AbstractRefreshableApplicationContext volatile DefaultListableBeanFactory beanFactory;
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
            // 3.准备使用上下文bean工厂基础信息
            // 配置工厂的标准上下文特征，例如上下文的类加载器和后处理器。（post-processors）
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
                // 4.允许在上下文子类中对bean工厂进行后处理；所有的bean已被加载，但未初始化
				postProcessBeanFactory(beanFactory);

				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
                // 5.实例化并调用bean工厂
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                // 6.注册bean处理器，拦截创建的bean做相应处理；保障bean处理器的顺序。（ArrayList,Comparator保障顺序）
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();

				// Initialize message source for this context.
                // 7.初始化上下文消息；下文未定义，使用父类级别。
				initMessageSource();

				// Initialize event multicaster for this context.
                // 8.为上下文初始化时间 多星；
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                // 9.在指定的上下文子类中初始化指定的beans，（默认为空）
				onRefresh();

				// Check for listener beans and register them.
                // 10.检查监听beans并且注册他们。即设置监听器Listeners
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                // 11.实例化所有剩余的（非lazy init）单例。
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
                // 12.最后一步：发布相应事件
				finishRefresh();
			} catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
				contextRefresh.end();
			}
		}
	}
```

```java
//spring 三级缓存 建议使用setter解决循环依赖问题 
//一二三级缓存Map实现 
//org.springframework.beans.factory.support.DefaultSingletonBeanRegistry 存放三个依赖位置
class DefaultSingletonBeanRegistry{
    /** Cache of singleton objects: bean name to bean instance. */
    //一级缓存 存放完整可用的bean对象 Object
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
    //三级缓存  存放创建中bean及其匿名内部类  ObjectFactory
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
    //二级缓存  存储创建中实例，AOP代理也在这一层 Object
	private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
}
```



### 2.Spring Aware接口

Aware：感知捕获的意思

**作用：辅助Spring bean访问Spring容器。**

- `BeanNameAware`：能够获取bean的名称，即是id
- `BeanFactoryAware`：获取BeanFactory实例
- `ApplicationContextAware`：获取`ApplicationContext`
- `MessageSourceAware`：获取MessageSource



























