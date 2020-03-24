# Spring学习

### 摘要

该部分涉及spring源码、spring boot、spring cloud各组件学习和理解。



### spring源码分析

#### BeanFactory：

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

#### org.springframework.beans.factory.support.DefaultListableBeanFactory

DefalutListableBeanFactory 为整个bean加载核心，是Spring注册及加载bean的默认实现



![DefaultListableBeanFactory](/Users/lzj11/Documents/DefaultListableBeanFactory.png)





org.springframework.core.AliasRegistry : 定义对alias的简单增删改等操作。
org.springframework.core.SimpleAliasRegistry ： 主要使用map作为alias的缓存，并对接口AliasRegistry进行实现。
。

。

。



org.springframework.beans.factory.xml.XmlBeanDefinitionReader















### springboot分析

#### spring boot-ssm







### springcloud分析

