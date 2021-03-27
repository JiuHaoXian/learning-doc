# JVM调优实战

> 在jvm中有很多的参数可以进行设置，这样可以让jvm在各种环境中都能够高效的运行。绝大部分的参数保持默认即可。



## JVM调优语法



### 三种参数类型

##### 标准参数

> ```-help```  ```-version```



##### 非标准参数（-X）

> ```-Xint```	```-Xcomp```
>
> jvm的-X参数是非标准参数，在不同版本的jvm中，参数可能会有所不同，可以通过java -X查看非标准参数。

```
-Xms与-Xmx分别是设置jvm的堆内存的初始大小和最大 大小。
-Xmx2048m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为2048M。
-Xms512m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为512M。
```



##### 非标准参数（-XX,使用率较高，不稳定）

主要用于jvm的调优和debug操作。适当的调整jvm的内存大小，可以充分利用服务器资源，让程序跑的更快。

- boolean类型

  > 语法：```-XX:[+-] ```表示启用或禁用属性
  >
  > 例子：```-XX:+DisableExplicitGC``` 表示禁用手动调用gc操作，也就是说调用System.gc()无效

- 非boolean类型

  > 语法：```-XX:=[value]```
  >
  > 例子：```	-XX:NewRatio=4	```表示新生代和老年代的比值为1:4

```shell
[root@node01 test]# java -Xms512m -Xmx2048m TestJVM
itcast

-XX:newSize
-XX:+UseSerialGC（使用某个GC） 
-XX:+PrintFlagsFinal(获取正在生效的设置)
```







### 常用命令

<<<<<<< HEAD
##### ```jstat```命令：查看堆内存各部分的使用量，以及加载类的数量
=======
##### ```jstat```命令：
>>>>>>> 4df259e0e3151344679740ab317073b7b8754dc3

```shell
#jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：
[root@node01 test] jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

F:\t>jstat -class 12076

Loaded Bytes     Unloaded  Bytes   Time
 5962  10814.2    0   0.0    3.75
 
#Loaded：加载class的数量
#Bytes：所占用空间大小
#Unloaded：未加载数量
#Bytes：未加载占用空间
#Time：时间
```

###### 查看编译统计

```shell
F:\t>jstat -compiler 12076

Compiled Failed Invalid  Time     FailedType FailedMethod
  3115    0     0        3.43     0
  
#Compiled：编译数量
#Failed：失败数量
#Invalid：不可用数量
#Time：时间
#FailedType：失败类型
#FailedMethod：失败的方法
```



##### 垃圾回收统计

```
<<<<<<< HEAD
S0C：第一个Survivor区的大小（KB）S1C：第二个Survivor区的大小（KB）S0U：第一个Survivor区的使用大小（KB）
S1U：第二个Survivor区的使用大小（KB）EC：Eden区的大小（KB）EU：Eden区的使用大小（KB）
OC：Old区大小（KB）OU：Old使用大小（KB）MC：方法区大小（KB）MU：方法区使用大小（KB）CCSC：压缩类空间大小（KB）
CCSU：压缩类空间使用大小（KB）YGC：年轻代垃圾回收次数YGCT：年轻代垃圾回收消耗时间FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间GCT：垃圾回收消耗总时间
```



=======
S0C：第一个Survivor区的大小（KB）
S1C：第二个Survivor区的大小（KB）
S0U：第一个Survivor区的使用大小（KB）
S1U：第二个Survivor区的使用大小（KB）
EC：Eden区的大小（KB）
EU：Eden区的使用大小（KB）
OC：Old区大小（KB）
OU：Old使用大小（KB）
MC：方法区大小（KB）
MU：方法区使用大小（KB）
CCSC：压缩类空间大小（KB）
CCSU：压缩类空间使用大小（KB）
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```



>>>>>>> 4df259e0e3151344679740ab317073b7b8754dc3
##### Jmap的使用以及内存溢出分析：

> 前面通过jstat可以对jvm堆的内存进行统计分析，而j==map可以获取到更加详细的内容==，如：==内存使用情况的汇总==、==对内存溢出的定位与分析==。

###### 查看内存使用情况

```
jmap -heap 6219
```

###### 查看内存中对象数量及大小

```shell
jmap -histo <pid> | more   #查看所有对象，包括活跃以及非活跃的

jmap -histo:live <pid> | more	#查看活跃对象 
```

###### 将内存使用情况dump到文件中

```shell
	jmap -dump:format=b,file=dumpFileName <pid>	#用法

	jmap -dump:format=b,file=/tmp/dump.dat 6219	#示例
```



##### 通过jhat对dump文件进行分析

```shell
#用法：

jhat -port <port> <file>

#示例：

[root@node01 tmp]# jhat -port 9999 /tmp/dump.dat 

Reading from /tmp/dump.dat...

Dump file created Mon Sep 10 01:04:21 CST 2018

Snapshot read, resolving...

Resolving 204094 objects...

Chasing references, expect 40 dots........................................

Eliminating duplicate references........................................

Snapshot resolved.

Started HTTP server on port 9999

Server is ready.
```



#### Jmp使用以及内存溢出分析

##### 使用MAT工具对内存溢出的定位与分析

> 内存溢出在实际的生产环境中经常会遇到，比如，不断的将数据写入到一个集合中，出现了死循环，读取超大的文件等等，都可能会造成内存溢出。

​		如果出现了内存溢出，首先我们需要定位到发生内存溢出的环节，并且进行分析，是正常还是非正常情况，如果是正常的需求，就应该考虑加大内存的设置，如果是非正常需求，那么就要对代码进行修改，修复这个bug。首先，我们得先学会如何定位问题，然后再进行分析。如何定位问题呢，我们需要借助于jmap与MAT工具进行定位分析。

当发生内存溢出时，会dump文件到java_pid5348.hprof。导入到MAT工具中进行分析



##### Jstack

> 有些时候我们需要查看下jvm中的线程执行情况，比如，发现服务器的CPU的负载突然增高了、出现了死锁、死循环等，我们该如何分析呢？

​		由于程序是正常运行的，没有任何的输出，从日志方面也看不出什么问题，所以就需要看下jvm的内部线程的执行情况，然后再进行分析查找出原因。

这个时候，就需要借助于jstack命令了，jstack的作用是将正在运行的jvm的线程情况进行快照，并且打印出来

```shell
#用法：jstack <pid>
[root@node01 bin]# jstack 2203
```



##### VisualVM工具的使用

> VisualVM，能够监控线程，内存情况，查看方法的CPU时间和内存中的对 象，已被GC的对象，反向查看分配的堆栈(如100个String对象分别由哪几个对象分配出来的)。VisualVM使用简单，几乎0配置，功能还是比较丰富的，几乎囊括了其它JDK自带命令的所有功能。

- 内存信息
- 线程信息
- Dump堆（本地进程
- Dump线程（本地进程）
- 打开堆Dump。堆Dump可以用jmap来生成
- 打开线程Dump
- 生成应用快照（包含内存信息、线程信息等等）
- 性能分析。CPU分析（各个方法调用时间，检查哪些方法耗时多），内存分析（各类对象占用的内存，检查哪些类占用内存多）
- ......



##### 监控远程JVM

> VisualVM不仅是可以监控本地jvm进程，还可以监控远程的jvm进程，需要借助于JMX技术实现。

监控Tomcat

使用VisualVM远程连接Tomcat



##### 可视化GC日志分析工具

1. ###### GC日志输出参数

> 前面通过-XX:+PrintGCDetails可以对GC日志进行打印，我们就可以在控制台查看，这样虽然可以查看GC的信息，但是并不直观，可以借助于第三方的GC日志分析工具进行查看。

在日志打印输出涉及到的参数如下：

```shell
-XX:+PrintGC 输出GC日志 

-XX:+PrintGCDetails 输出GC的详细日志 

-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式） 

-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800） 

-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息 

-Xloggc:../logs/gc.log 日志文件的输出路径 
```

测试：

```shell
-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -Xmx256m -XX:+PrintGCDetails 

-XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC

-Xloggc:F://test//gc.log 
```

运行后就可以在E盘下生成gc.log文件。

1. 使用GC Easy（可视化）

1. PerfMa(分析工具)



#### 对于**JVM**调优的建议：

- 生产环境的JVM一定要进行参数设定，不能全部默认上生产。
- 对于参数的设定，不能拍脑袋，需要通过实际并发情况或压力测试得出结论。
- 对于内存中对象临时存在居多的情况，将年轻代调大一些。如果是G1或ZGC，不需要设定。
- 仔细分析gceasy给出的报告，从中分析原因，找出问题。
- 对于低延迟的应用建议使用G1或ZGC垃圾收集器。
- 不要将焦点全部聚焦jvm参数上，影响性能的因素有很多，比如：操作系统、tomcat本身的参数等。



#### Tomcat8优化

1. ##### 禁用AJP连接

> AJP（Apache JServer Protocol） AJPv13协议是面向包的。WEB服务器和Servlet容器通过TCP连接来交互；为了节省SOCKET创建的昂贵代价，WEB服务器会尝试维护一个永久TCP连接到servlet容器，并且在多个请求和响应周期过程会重用连接。

我们一般是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用。

修改conf下的server.xml文件，将AJP服务禁用掉即可。

```shell
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

2. ##### 执行器（线程池）

在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。

3. ##### 三种运行模式

   > tomcat的运行模式有3种：

   * bio:  默认的模式,性能非常低下,没有经过任何优化处理和支持

   * nio nio(new I/O):  是Java SE 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。

   * apr :  安装起来最困难,但是从操作系统级别来解决异步的IO问题,大幅度的提高性能.

   > 推荐使用nio，不过，在tomcat8中有最新的nio2，速度更快，建议使用nio2.

4. ##### 设置nio2

```shell
	<Connector executor="tomcatThreadPool" port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" connectionTimeout="20000" 		redirectPort="8443" />
```



## 代码优化建议

1. 尽可能使用局部变量。
2. 尽量减少对变量的重复计算
3. 尽量采用蓝加载的策略，即在需要的时候才创建
4. 异常不应该用来控制流程
5. 不要将数组声明为public static final
6. 不要创建一些不使用的对象，不要导入一些不使用的类
7. 程序运行过程中避免使用反射
8. 使用数据库连接池和线程池
9. 容器初始化尽可能指定长度
10. ArrayList随机遍历快，LinkedList添加删除快
11. 使用Entry遍历map
12. 不要手动调用System().gc;
13. String尽量少用正则表达式
14. 日志的输出要注意级别
15. 对资源的close()建议分开操作