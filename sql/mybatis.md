# mybatis使用分析和问题处理

### 问题篇

#### 1.mybatis包通用测试插入数据出现重复情况

场景描述：
工具：IDEA

工程基础：springboot的SSM基础创建

问题原因：IDEA设置问题

解决办法：

​	idea的settings中 build->maven->runner右侧将**delegate IDEA去掉**
参考依据：https://segmentfault.com/q/1010000018970005 回答部分



####  2.mybatis关于通用 Mapper 中 selectByPrimaryKey 无法根据id查询

问题描述：
数据库表数据主键bigint；
mybatis的mapper中为long型；查询selectByPrimaryKey 是出错。

解决办法：
mybatis的mapper中为**long型改为Long**。

参考依据：https://blog.csdn.net/qq_37119719/article/details/85250496



### mybatis分析(基于3.4.6版本)

#### 1.mybatis独立使用步骤

1. **建立PO**

2. **建立Mapper**--->（dao层）

3. **建立配置文件**

   spring boot mybatis通用jar在tk.mybatis.mapper.autoconfigure.MybatisProperties对应配置属性；详见源码和《Spring源码深度解析（第2版）.pdf》mybatis篇章

4. **创建映射文件**

5. **创建单元测试类**



#### 2.mybatis源码分析

**1.org.apache.ibatis.session.SqlSessionFactory---设置mybatis请求共享到Spring application的context中。**

spring通过**org.mybatis.spring.SqlSessionFactoryBean**封装了Mybatis中的实现。



**2.org.mybatis.spring.mapper.MapperFactoryBean---设置mybatis映射端口**



**3.org.mybatis.spring.mapper.MapperScannerConfigurer---进行包扫描配置**











