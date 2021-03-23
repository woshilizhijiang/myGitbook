# Spring学习

### 摘要

该部分涉及spring源码、spring boot、spring cloud各组件学习和理解。



## 基础问题

### spring

#### Qualifier注解





### 一、spring源码分析

#### 1.BeanFactory

提供IOC和依赖注入

org.springframework.beans.factory.BeanFactory：提供Factory模式的实现来消除对**程序性单例模式**的需要。

源码分析：

```java
package org.springframework.beans.factory;

import org.springframework.beans.BeansException;
import org.springframework.core.ResolvableType;
import org.springframework.lang.Nullable;
public interface BeanFactory {

	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
  
	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

	String[] getAliases(String name);

}

```



核心类：

#### 2.DefaultListableBeanFactory

DefalutListableBeanFactory 为整个bean加载核心，是Spring注册及加载bean的默认实现







org.springframework.core.AliasRegistry : 定义对alias的简单增删改等操作。
org.springframework.core.SimpleAliasRegistry ： 主要使用map作为alias的缓存，并对接口AliasRegistry进行实现。
。

。

org.springframework.beans.factory.xml.XmlBeanDefinitionReader



#### 3.spring容器的基础XmlBeanFactory

##### 配置文件封装

Spring加载文件抽象类Resource：UML图如下：




##### 加载Bean



#### 4.spring循环依赖问题

**1.构造器参数循环依赖**

表示通过构造器注入构成的循环依赖，此依赖是无法解决的，只能抛出**BeanCurrentlyIn CreationException**异常表示循环依赖。

**2.setter方式单例，默认方法**
bean实例化过程

![image-20200512153151778](/Users/lzj11/Library/Application%20Support/typora-user-images/image-20200512153151778.png)



**3.setter方式原型，prototype**

对于"prototype"作用域bean，Spring容器无法完成依赖注入，因为Spring容器不进行缓存"prototype"作用域的bean，因此无法提前暴露一个创建中的bean。

Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: 
  Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference? 

**为什么原型模式就报错了呢 ？**

对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容
器不进行缓存，因此无法提前暴露一个创建中的Bean。



### springboot分析



#### spring boot-ssm







### springcloud分析





