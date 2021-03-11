# 多线程（JUC）

> 在 Java 5.0 提供了 java.util.concurrent （简称JUC ）包,在此包中增加了在并发编程中很常用的实用工具类，用于定义类似于线程的自定义子系统，包括线程池、异步 IO 和轻量级任务框架。提供可调的、灵活的线程池。还提供了设计用于多线程上下文中的 Collection 实现等。





## 多线程基础

##### 多线程的创建方式？

* 继承`Thread`方法，通过`new MyThread().start();`进行调用。

* 实现`Runnable`接口，通过`new Thread(new MyThread()).start();`调用。

* `Executors.newCachedThreadPool`创建一个线程池。



##### 多线程的常用方法？

* `Thread.Yield();`	让出当前CPU，进入CPU等待队列，让其他线程先执行。

* `Thread.Join();`	常用来等待另一个线程的结束

* `getState();`	获取线程状态



#### 线程状态的迁移？

总共分为6个状态，new、Runnable、Blocked、Waiting、TimedWaiting、Teminated

![image-20210307032724553](JUC-%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8E%E9%AB%98%E5%B9%B6%E5%8F%91.assets/image-20210307032724553.png)

> `Runnable`？

​		当``new一个线程` `start()`之后，会交给**线程调度器执行**此时的状态叫`runnable`，在这里会有2种状态，一个是在等待队列里等待执行时的`Ready`状态，另一个是执行中`Running`状态，当线程被线程调度器选中执行时，会变成`Running`状态，如果在`Running`时调用了`Thread.Yield();`则会回到`Ready`状态，如果执行结束进入`Teminated`结束状态，在此状态后不允许在此进入`new`调用`start().`



> `Waiting`？

当在运行中调用了`Wait()`,`join()`,`LockSupport.park()`进入`Waiting`状态，调用`notify()`、`notifyAll`、`LockSupport.unpark()`进入`Runnable`的`Running`状态



> `TimedWaiting`?

`sleep(time)`、`join(time)`、`wait(time)`等等 进入此状态，时间结束自动回到`TimedWaiting`。



> `Blocked`?

未获得锁时 进入，获得锁后 离开。 



## JAVA锁机制

### CAS （Compare And Set）无锁优化-自旋

> CAS算法是计算机硬件对并发操作共享数据的支持，CAS包含3个操作数：

* CAS算法：内存值V，预估值A，更新值B

* ABA问题
  * 值类型没有问题，对象（引用类型）的话就会出问题





### Synchronized

> Synchronized关键字，对某个对象（Object）加锁 。当对某个对象加锁后，必须要拿到这个对象的锁才能继续访问。

#### 特点：

* 可重入锁、独占锁
* 不能对`String`、`Integer`、`Long`
* 锁定方法与非锁定方法可同时执行
* 既保证了原子性又保证了可见性
* 程序中若出现异常，锁将会被释放
* 锁只能升级不能降级。
* 执行时间长、数量多的用系统（OS）锁，执行时间短、线程少的用自旋锁
* 锁的属性发生改变，没有问题，当重新赋值或new后则出现问题，加finol
* synchronized不可响应中断，一个线程获取不到锁就一直等着



#### synchronized锁升级

> 偏向锁-自旋锁-重量级锁

JDK早期是重量级的，需要向OS去申请，JDK1.5之后，后来改进成锁升级的一个概念，

当使用sync时，先是无锁状态，然后在markword记录这个线程的ID（偏向锁）一个线程在执行

如果有线程争用，升级为自旋锁，10次

10次后，升级为重量级锁。









### Volatile

* 保证线程的可见性
  * MESI
  * CPU缓存一执性协议

* 禁止指令重排序（CPU）：

  > 模型里有8个指令完成数据的读写，通过其中load和store指令相互组合成4个内存屏障实现禁止指令重排序。
  >
  >  JVM new一个对象时主要有三部指令：
  >
  > 1.申请一个内存
  >
  > 2.给成员变量赋初始值
  >
  > 3.三是将指针付给定义

  * DCL单例
  * Double Check Lock
  * Mgr06.java
    * loadfence原语指令
    * storefence原语指令



### ReentrantLock(重入锁)

* 独占、可重入锁
* 加锁解锁手动进行，且次数需要一样
* 锁可以中断
* ReentrantLock还可以实现公平锁机制



### AbstractQueuedSynchronizer（AQS）

> 抽象的队列式的同步器 ：AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch...。

![img](/Users/gaoshengwang/%E8%BD%AF%E4%BB%B6%E6%8A%80%E6%9C%AF/%E6%8A%80%E6%9C%AF%E6%96%87%E6%A1%A3/java/images/721070-20170504110246211-10684485.png)

概括：它维护了一个**volatile int state**（代表公共资源）和一个**FIFO线程等待队列**（多线程争用资源被阻塞时会进入此队列）



#### state的访问方式有三种

* getState()
* setState()
* compareAndSetState() 它定义两种资源共享方式
  * Exclusive 独占
  * Share 共享





### 读写锁（ReadWriteLock）