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









