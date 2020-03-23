# mysql-技术内幕Innodb



### chapter 4 表-table

#### 4.1索引组织表（index organized table）







### 第五章 索引和算法

#### 5.1InnoDB存储引擎索引概述

B+树索引

全文索引

哈希索引


#### 5.5Cardinality值（基数）

##### B+树索引范围

低选择性：无需用B+树索引；字段如性别、地区、类型这些范围小的属性
高选择性：适合用B+树索引；如姓名这类重复性低字段属性

##### 索引选择性高低查看

show index	

##### cardinality的随机采样









