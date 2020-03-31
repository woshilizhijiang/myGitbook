# Groovy指南

### 下载安装(mac 版本)

https://groovy.apache.org/download.html

```groovy
wget https://dl.bintray.com/groovy/maven/apache-groovy-sdk-3.0.1.zip

cd ~
vi .bash_profile
export PATH=$PATH:/Users/lzj11/Downloads/groovy-3.0.1/bin  

source ~/.bash_profile

groovy -v
```



### 基本语法

- 变量的类型：基本类型（java中的int,float,double,byte,char,long,short）和对象类型(String等) （Groovy中最终都是对象类型）
- 变量的定义：强类型定义方式和弱类型def定义方式
- 强类型定义方式和弱类型def定义方式的选择

```java
roovy与Java集成常见的坑
摘要： groovy特性 Groovy是一门基于JVM的动态语言，同时也是一门面向对象的语言，语法上和Java非常相似。它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。 Java作为一种通用、静态类型的编译型语
groovy特性
Groovy是一门基于JVM的动态语言，同时也是一门面向对象的语言，语法上和Java非常相似。它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。
Java作为一种通用、静态类型的编译型语言有很多优势，但同样存在一些负担：
  ● 重新编译太费工；
  ● 静态类型不够灵活，重构起来时间可能比较长；
  ● 部署的动静太大；
  ● java的语法天然不适用生产dsl；
相对于Java，它在编写代码的灵活性上有非常明显的提升，对于一个长期使用Java的开发者来说，使用Groovy时能够明显地感受到负身上的“枷锁”轻了。Groovy是动态编译语言，广泛用作脚本语言和快速原型语言，主要优势之一就是它的生产力。Groovy 代码通常要比 Java 代码更容易编写，而且编写起来也更快，这使得它有足够的资格成为开发工作包中的一个附件。
Java不是解决动态层问题的理想语言，这些动态层问题包括原型设计、脚本处理等。可以把Groovy看作给Java静态世界补充动态能力的语言，同时Groovy已经实现了java不具备的语言特性：
  ● 函数字面值；
  ● 对集合的一等支持；
  ● 对正则表达式的一等支持；
  ● 对xml的一等支持；
groovy与java集成的方式
重温下Groovy调用Java方式，包括使用GroovyClassLoader、GroovyShell和GroovyScriptEngine。
GroovyClassLoader
用 Groovy 的 GroovyClassLoader ，动态地加载一个脚本并执行它的行为。GroovyClassLoader是一个定制的类装载器，负责解释加载Java类中用到的Groovy类。
GroovyClassLoader loader = new GroovyClassLoader();
Class groovyClass = loader.parseClass(new File(groovyFileName));
GroovyObject groovyObject = (GroovyObject) groovyClass.newInstance();
groovyObject.invokeMethod("run", "helloworld");
GroovyShell
GroovyShell允许在Java类中（甚至Groovy类）求任意Groovy表达式的值。您可使用Binding对象输入参数给表达式，并最终通过GroovyShell返回Groovy表达式的计算结果。
GroovyShell shell = new GroovyShell();
Script groovyScript = shell.parse(new File(groovyFileName));
Object[] args = {};
groovyScript.invokeMethod("run", args);
GroovyScriptEngine
GroovyShell多用于推求对立的脚本或表达式，如果换成相互关联的多个脚本，使用GroovyScriptEngine会更好些。GroovyScriptEngine从您指定的位置（文件系统，URL，数据库，等等）加载Groovy脚本，并且随着脚本变化而重新加载它们。如同GroovyShell一样，GroovyScriptEngine也允许您传入参数值，并能返回脚本的值。
Groovy代码文件与class文件的对应关系
而作为基于JVM的语言，Groovy可以非常容易的和Java进行互操作，但也需要编译成class文件后才能运行，所以了解Groovy代码文件和class文件的对应关系，有助于更好地理解Groovy的运行方式和结构。
对于没有任何类定义
如果Groovy脚本文件里只有执行代码，没有定义任何类（class），则编译器会生成一个Script的子类，类名和脚本文件的文件名一样，而脚本的代码会被包含在一个名为run的方法中，同时还会生成一个main方法，作为整个脚本的入口。
对于仅有一个类
如果Groovy脚本文件里仅含有一个类，而这个类的名字又和脚本文件的名字一致，这种情况下就和Java是一样的，即生成与所定义的类一致的class文件。
对于多个类
如果Groovy脚本文件含有多个类，groovy编译器会很乐意地为每个类生成一个对应的class文件。如果想直接执行这个脚本，则脚本里的第一个类必须有一个static的main方法。
groovy与java集成中经常出现的问题
使用GroovyShell的parse方法导致perm区爆满的问题
如果应用中内嵌Groovy引擎，会动态执行传入的表达式并返回执行结果，而Groovy每执行一次脚本，都会生成一个脚本对应的class对象，并new一个InnerLoader去加载这个对象，而InnerLoader和脚本对象都无法在gc的时候被回收运行一段时间后将perm占满，一直触发fullgc。
  ● 为什么Groovy每执行一次脚本，都会生成一个脚本对应的class对象？
一个ClassLoader对于同一个名字的类只能加载一次，都由GroovyClassLoader加载，那么当一个脚本里定义了C这个类之后，另外一个脚本再定义一个C类的话，GroovyClassLoader就无法加载了。为什么这里会每次执行都会加载？
这是因为对于同一个groovy脚本，groovy执行引擎都会不同的命名，且命名与时间戳有关系。当传入text时，class对象的命名规则为："script" + System.currentTimeMillis() + Math.abs(text.hashCode()) + ".groovy"。这就导致就算groovy脚本未发生任何变化，每次执行parse方法都会新生成一个脚本对应的class对象，且由GroovyClassLoader进行加载，不断增大perm区。
  ● 为什么InnerLoader加载的对应无法通过gc清理掉？
大家都知道，JVM中的Class只有满足以下三个条件，才能被GC回收，也就是该Class被卸载：1. 该类所有的实例都已经被GC，也就是JVM中不存在该Class的任何实例；2. 加载该类的ClassLoader已经被GC；3. 该类的java.lang.Class对象没有在任何地方被引用，如不能在任何地方通过反射访问该类的方法。
在GroovyClassLoader代码中有一个class对象的缓存，进一步跟下去，发现每次编译脚本时都会在Map中缓存这个对象，即：setClassCacheEntry(clazz)。每次groovy编译脚本后，都会缓存该脚本的Class对象，下次编译该脚本时，会优先从缓存中读取，这样节省掉编译的时间。这个缓存的Map由GroovyClassLoader持有，key是脚本的类名，这就导致每个脚本对应的class对象都存在引用，无法被gc清理掉。
  ● 如何解决？
请参考：Groovy引发的PermGen区爆满问题定位与解决。
如需更深入的理解GroovyClassLoader体系，请参考下面这篇文章Groovy深入探索——Groovy的ClassLoader体系
使用GroovyClassLoader加载机制导致频繁gc问题
通常使用如下代码在Java 中执行 Groovy 脚本：
GroovyClassLoader groovyLoader = new GroovyClassLoader();
Class<Script> groovyClass = (Class<Script>) groovyLoader.parseClass(groovyScript);
Script groovyScript = groovyClass.newInstance();
每次执行groovyLoader.parseClass(groovyScript)，Groovy 为了保证每次执行的都是新的脚本内容，会每次生成一个新名字的Class文件，这个点已经在前文中说明过。当对同一段脚本每次都执行这个方法时，会导致的现象就是装载的Class会越来越多，从而导致PermGen被用满。
同时这里也存在性能瓶颈问题，如果去分析这段代码会发现90%的耗时占用在Class
为了避免这一问题通常做法是缓存Script对象，从而避免以上2个问题。在这过程中通常又会引入新的问题：
  ● 高并发情况下，binding对象混乱导致计算出错
在高并发的情况下，在执行赋值binding对象后，真正执行run操作时，拿到的binding对象可能是其它线程赋值的对象，所以出现数据计算混乱的情况
  ● 长时间运行仍然出现oom，无法解决Class
这点在上文中已经提到，由于groovyClassLoader会缓存每次编译groovy脚本的Class对象，下次编译该脚本时，会优先从缓存中读取，这样节省掉编译的时间。导致被加载的Class对象因为存在引用而无法被卸载，虽然通过缓存避免了短时间内大量生成新的class对象，但如果长时间运营仍然会存在问题。
比较好的做法是：
  ● 每个 script 都 new 一个 GroovyClassLoader 来装载；
  ● 对于 parseClass 后生成的 Class 对象进行cache，key 为 groovyScript 脚本的md5值。
CodeCache用满，导致JIT禁用问题
对于大量使用Groovy的应用，尤其是 Groovy 脚本还会经常更新的应用，由于这些Groovy脚本在执行了很多次后都会被JVM编译为 native 进行优化，会占据一些 CodeCache 空间，而如果这样的脚本很多的话，可能会导致 CodeCache 被用满，而 CodeCache 一旦被用满，JVM 的 Compiler 就会被禁用，那性能下降的就不是一点点了
```


































