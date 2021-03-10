## Python

```python
#python遍历list方法：
#1.for直接遍历列表
#https://blog.csdn.net/sinat_29675423/article/details/87972498

list=[2,3,4]
for num in list:
    print(num)
    
#2.利用python内置enumerate列举list中的数
list=[2,3,4]
for num in enumerate(list):
    print(num)

#3.使用iter（）迭代器
list=[2,3,4]
for num in iter(list):
    print(num)

#4.使用range()函数
list=[2,3,4]
for num in range(len(list)):
    print i,list[i]
```



```python
1.bytes，str相互转换
bytes 转换为 str
str(b, encoding = "utf-8")  
str(b, encoding = "gbk")  

import chardet
ret = chardet.detect(b)

str 转换为 bytes
b=bytes(str1, encoding='utf-8')
print(b)
b=str1.encode('utf-8')
print(b)
```



```python
#基础语法

标识符
第一个字符必须是字母表中字母或下划线 _ 。
标识符的其他的部分由字母、数字和下划线组成。
标识符对大小写敏感。
在 Python 3 中，可以用中文作为变量名，非 ASCII 标识符也是允许的了。

保留字
import keyword
>>> keyword.kwlist
['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']

注释
单行注释以 # 开头
多行注释可以用多个 # 号，还有 ''' 和 """：

行与缩进
python最具特色的就是使用缩进来表示代码块，不需要使用大括号 {}
缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数

多行语句
一行写完一条语句，但如果语句很长，我们可以使用反斜杠(\)来实现多行语句
total = item_one + \
        item_two + \
        item_three
在 [], {}, 或 () 中的多行语句，不需要使用反斜杠(\)，例如：
total = ['item_one', 'item_two', 'item_three',
        'item_four', 'item_five']

数值类型
整数、布尔型、浮点数、复数

字符串
单引号、双引号都可代表
#使用三引号('''或""")可以指定一个多行字符串。
#转义符 '\'
反斜杠可以用来转义，使用r可以让反斜杠不发生转义。。 如 r"this is a line with \n" 则\n会显示，并不是换行。
字符串可以用 + 运算符连接在一起，用 * 运算符重复
Python 中的字符串有两种索引方式，从左往右以 0 开始，从右往左以 -1 开始。
Python中的字符串不能改变


同一行显示多条语句
用;分隔

import 或者 from...import 来导入相应的模块

python6个标准数据类型:
Number（数字）
String（字符串）
List（列表）
Tuple（元组）
Set（集合）
Dictionary（字典）

#Number
isinstance 和 type 的区别在于：
type()不会认为子类是一种父类类型。
isinstance()会认为子类是一种父类类型。

#基础运算
+
-
*
/  ........除法得到一个浮点数
//  ........除法得到一个整数
% 
** 乘方

#字符串
单双引号
r屏蔽反斜杠发生转义

#List
List（列表） 是 Python 中使用最频繁的数据类型。
列表可以完成大多数集合类的数据结构实现。列表中元素的类型可以不相同，它支持数字，字符串甚至可以包含列表（所谓嵌套）。
列表是写在方括号 [] 之间、用逗号分隔开的元素列表。

#Tuple（元组）
元组（tuple）与列表类似，不同之处在于元组的元素不能修改。元组写在小括号 () 里，元素之间用逗号隔开。

#Set（集合）
集合（set）是由一个或数个形态各异的大小整体组成的，构成集合的事物或对象称作元素或是成员。
基本功能是进行成员关系测试和删除重复元素。
可以使用大括号 { } 或者 set() 函数创建集合，注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。


#Dictionary（字典）
字典（dictionary）是Python中另一个非常有用的内置数据类型。
列表是有序的对象集合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典是一种映射类型，字典用 { } 标识，它是一个无序的 键(key) : 值(value) 的集合。
键(key)必须使用不可变类型。
在同一个字典中，键(key)必须是唯一的。

##python str和int相互转换
a=1
b='1'
b == str(a)
a == int(b)
注：参考网站：https://www.runoob.com/python3/python3-data-type.html

#解释器
python3 HelloWorld.py

#迭代
迭代是Python最强大的功能之一，是访问集合元素的一种方式。
迭代器是一个可以记住遍历的位置的对象。
迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退。
迭代器有两个基本的方法：iter() 和 next()。
字符串，列表或元组对象都可用于创建迭代器：

#函数
def functionName():
    print("name")
必需参数functionName(myName) //顺序必须一致，类似java正常传入参数
关键字参数functionName(myName=name) //不要求参数顺序
默认参数 functionName(age=15)
不定长参数 functionName(*params)//泛型数组参数
关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。
使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值。

#匿名函数
print("匿名函数")
sum = lambda arg1,arg2:arg1+arg2 //形似三目表达式
print(sum(2,6))

#isin函数


#~ ： 按位取反
```

### 语法

- multiprocessing ---基于进程的并行

- ```python
  from multiprocessing import Pool
  def f(x):
      retrun x * x
  if __name__ =='__main__':
      with Pool(5) as p:
          print(p.map(f,[1,2,3]))
     
  ```

  





### 1.pandas用法

```python
1、pd.set_option(‘expand_frame_repr’, False)
True就是可以换行显示。设置成False的时候不允许换行

2、pd.set_option(‘display.max_rows’, 10)
pd.set_option(‘display.max_columns’, 10)
显示的最大行数和列数，如果超额就显示省略号，这个指的是多少个dataFrame的列。如果比较多又不允许换行，就会显得很乱。

3、pd.set_option(‘precision’, 5)
显示小数点后的位数

4、pd.set_option(‘large_repr’, A)
truncate表示截断，info表示查看信息，一般选truncate

5、pd.set_option(‘max_colwidth’, 5)
列长度

6、pd.set_option(‘chop_threshold’, 0.5)
绝对值小于0.5的显示0.0

7、pd.set_option(‘colheader_justify’, ‘left’)
显示居中还是左边，

8、pd.set_option(‘display.width’, 200)
横向最多显示多少个字符， 一般80不适合横向的屏幕，平时多用200.
```

