## Linux



### 1.账号授信

```shell




```



### 2.网卡

```shell
#Linux下多网卡绑定bond及模式介绍
Linux下一共有七种网卡bond方式，实现以上某个或某几个具体功能。
最常见的三种模式是bond0，bond1，bond6.

【bond0】
平衡轮循环策略，有自动备援，不过需要"Switch"支援及设定。
balance-rr（Round-robin policy）   
方式：
传输数据包的顺序是依次传输（即：第一个包走eth0，第二个包就走eth1……，一直到所有的数据包传输完成）。
优点：
提供负载均衡和容错能力。
缺点：
同一个链接或者会话的数据包从不同的接口发出的话，中间会经过不同的链路，在客户端可能会出现数据包无法有序到达的情况，而无序到达的数据包将会被要求重新发送，网络吞吐量反而会下降。

【bond1】
主-备份策略
active-backup（Active -backup policy）
方式：
只有一个设备处于活动状态，一个宕掉之后另一个马上切换为主设备。
mac地址为外部可见，从外面看，bond的mac地址是唯一的，switch不会发生混乱。
优点：
提高了网络连接的可靠性。
缺点：
此模式只提供容错能力，资源利用性较低，只有一个接口处于active状态，在有N个网络接口bond的状态下，利用率只有1/N。

【bond2】
平衡策略
balance-xor（XOR policy）
方式：
基于特性的Hash算法传输数据包。
缺省的策略为：(源MAC地址 XOR 目标MAC地址) % slave数量。 # XRO为异或运算，值不同时结果为1，相同为0
可以通过xmit_hash_policy选项设置传输策略。
特点：
提供负载均衡和容错能力。

【bond3】
广播策略
broadcast
方式：
在每个slave接口上传输每一个数据包。
特点：
提供容错能力。

【bond4】
IEEE 802.3ad 动态链接聚合
802.3ad（ IEEE 802.3ad Dynamic link aggregation）
方式：
创建一个聚合组，共享同样的速率和双工设定。
根据802.3ad规范将多个slave工作在同一个激活的聚合体下。外出流量的slave选举基于传输Hash策略，同样，此策略也可以通过xmit_hash_policy选项进行修改。
注意：
并不是所有的传输策略都是802.3ad所适应的。
条件：
1. ethtool支持获取每个slave的速率和双工设定。
2. switch支持IEEE 802.3ad Dynamic link aggregation（大多数交换机需要设定才支持）
  
【bond5】
适配器传输负载均衡
balance-tlb（Adaptive transmit load balancing）
方式：
在每个slave上根据当前的负载（依据速度）分配外出流量，接收时使用当前轮到的slave。
如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。
条件：
ethtool支持获取每个slave的速率。
特点：
不需要任何特别的switch(交换机)支持的通道bonding。

【bond6】
适配器适应性负载均衡
balance-alb（Adaptive load balancing） 
方式：
此模式包含了bond5的balance-tlb，同时增加了针对IPV4流量的接收负载均衡。（receive load balance， rlb）
接收负载均衡是通过ARP协商实现的。
bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。
来自服务器端的接收流量也会被均衡。
当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。
当ARP应答从对端到达时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。
使用ARP协商进行负载均衡的一个问题是：
每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。
这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。
当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。
接收的负载被顺序地分布（round robin）在bond中最高速的slave上当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。
下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答不会被switch(交换机)阻截。
条件：
1. ethtool支持获取每个slave的速率
2. 底层驱动支持设置某个设备的硬件地址
特点：
总是有一个slave（curr_active_slave）使用bond的硬件地址，同时每个bond里面的slave都有一个唯一的硬件地址。
如果curr_active_slave出了故障，则它的硬件地址会被重新选举产生的slave接管。
与bond0最大的区别在于，bond0的多张网卡里面的流量几乎是相同的，但是bond6里面的流量是先占满eth0，再占满eth1……依次
```

```shell
#1.shell 单引号和双引号区别:双引号变量会被展开、单引号变量不再展开
a="abc"
echo "str=$a" #结果显示str=abc
echo 'str=$a' #结果显示str=$a

#2.shell 两种执行方式
chmod +x ./test.sh #使脚本具有执行权限
./test.sh #执行脚本

/bin/sh test.sh

#3.awk是一种编程语言，用于linux/unix下对文本和数据进行扫描与处理。数据可以来自标准输入、文件、管道。

#4.逻辑if语句
if [ command ];then
	符合该条件执行的语句
elif [ command ];then
	符合该条件执行的语句
else
	符合该条件执行的语句
if


#5 shell 脚本中$$,$#,$?分别代表什么意思?
$0 这个程式的执行名字
$n 这个程式的第n个参数值，n=1..9
$* 这个程式的所有参数,此选项参数可超过9个。
$# 这个程式的参数个数
$$ 这个程式的PID(脚本运行的当前进程ID号)
$! 执行上一个背景指令的PID(后台运行的最后一个进程的进程ID号)
$? 执行上一个指令的返回值 (显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误)
$- 显示shell使用的当前选项，与set命令功能相同
$@ 跟$*类似，但是可以当作数组用

#6.shell中的X的用法
#(命令列第一个参数) $1 如果只等如X, 那就是变量没有内容,是空变量, 也
#是用来测试命令列上有没有参数,例如
user@minix-nb:~$ cat a
#! /bin/bash
if [ X$1 = X ]
    then
      echo "the first argu is empty"
    else
      echo "the first argu is $1"
fi
user@minix-nb:~$ ./a
the first argu is empty
user@minix-nb:~$ ./a 123
the first argu is 123
user@minix-nb:~$
```

