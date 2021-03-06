### 集合类型

集合类型主要有3种：set(集）、list(列表）和map(映射)。

集合接口分为：Collection和Map

list、set实现了Collection接口



#### 常见问题

##### 集合类取并集取交集？

```java
    List<String> intersection = list1.stream().filter(item -> list2.contains(item)).collect(toList());
    System.out.println("---交集 intersection---");
    intersection.parallelStream().forEach(System.out :: println);
```



##### jdk1.7与jdk1.8中HashMap区别

1. 1.7是数组+链表，1.8是数组+链表+红黑树
2. jdk1.7中当哈希表为空时，会先调用inflateTable()初始化一个数组；而1.8则是直接调用resize()扩容;
3. 1.7是在头部插入，1.8是尾插
4. 1.7是hash值是直接hashCode，1.8是无符号右移+异或运算，保留了高低位特征，使元素更均匀，降低哈希碰撞。
5. 扩容时1.8会保持原链表的顺序，而1.7会颠倒链表的顺序；而且1.8是在元素插入后检测是否需要扩容，1.7则是在元素插入前；
6.  jdk1.8是扩容时通过hash&cap==0将链表分散，无需改变hash值，而1.7是通过更新hashSeed来修改hash值达到分散的目的；
7. 1.7中是只要不小于阈值就直接扩容2倍；而1.8的扩容策略会更优化，当数组容量未达到64时，以2倍进行扩容，超过64之后若桶中元素个数不小于7就将链表转换为红黑树，但如果红黑树中的元素个数小于6就会还原为链表，当红黑树中元素不小于32的时候才会再次扩容。

##### hashmap和hashlist的扩容机制？

arrayList：默认10，扩容为1.5倍

hashmap：16，0.75时扩容，2倍







#### 常用集合的分类：

### List

> 可以重复，通过索引取出加入数据，顺序与插入顺序一致，元素可以为NULL

* ArrayList： （**==数组==、不安全**）
  * **数组**  结构array，查询速度快，增删改慢。
  * 默认大小为  **10 ** 扩容为增加当前容量的50%。
  * 线程不安全
  * 随机访问速度极快，时间复杂度是O(1)。



* Vector：（**数组、安全**）向量
  * **数组**  结构array，与ArrayList相同，查询速度快，增删改慢；

  * 扩容为增加当前容量的100%。

  * 线程安全（synchronized）

  * 随机访问速度极快，时间复杂度是O(1)。

    * CopeOnWriteArrayList：（实现List接口、线程安全、没扩容的概念）除了使用Vector，我们可以使用java.io.concurrent包下的CopeOnWriteArrayList，他是一个线程安全的List，底层是通过复制数组实现的，为什么安全呢，在他的add方法里首先它会加lock锁，锁住后复制一个新的数组，然后加入到新数组里，最后把对象的指针改为指到新的数组，这里和文件系统的cow很类似

      * CopeOnWrite机制

        * 在Linux中进程都是被fork出来的，除了进程号，fork出来的子进程和父进程是一模一样的，当使用了CopeOnWrite机制，这两个进程用的是同一个内存空间，当父进程有修改发生时，再为子进程分配相应的物理空间。

          这样做的好处就是为了处理复制大量资源时带来的瞬时延迟，----懒汉模式或懒加载

        * 文件系统中其实也有cow机制。在文件系统中修改数据库A的时候其实会把A读出来，写到B去，这样的好处是防止在A修改的过程中断电，原来的A还在，保证了数据的完整性

      * 缺点：消耗内存，只能保证数据的最终一致性，不能保证实时一致性



* LinkedList： （**==链表==、不安全**）

  *  **链表**  结构，增删速度快，查询稍慢；指针指向后一个元素
  
  

### Set

> 数据无序且唯一，实现类都不是线程安全的类，解决方案：使用sysnchronizedSet对象
>
> `Set  set = Collections.sysnchronizedSet`

* HashSet：（**哈希表、无序、不可重复、不安全、可以为空**）
  * 通过哈希表（散列/hash）实现，是Set接口（Set接口是继承了Collection接口的）最常用的实现类
  * 其底层其实也是一个数组，存在的意义是提供查询速度，插入的速度也是比较快，但是适用于少量数据的插入操作
  * 判断两个对象是否相等的规则：
    * 1. equals比较为true；
    * 2. hashCode值相同。要求：要求存在在哈希表中的对象元素都得覆盖equals和hashCode方法。
    
  * LinkedHashSet：
    * 继承了HashSet类，==哈希表+链表==的数据结构，但因为多加了一种数据结构，所以效率较低，不建议使用，如果要求一个集合既要保证元素不重复，也需要记录元素的先后添加顺序，才选择使用LinkedHashSet



* TreeSet：（**==红黑树==、有序、不可重复、安全、不可以为空**）
  * Set接口的实现类，也拥有set接口的一般特性，但是不同的是他也实现了SortSet接口，底层通过**红黑树**（Red-Black tree）
    * 红黑树：红黑树就是满足一下红黑性质的二叉搜索树
      * ①每个节点是黑色或者红色
      * ②根节点是黑色的
      * ③每个叶子结点是黑色的
      * ④如果一个节点是红色的，那么他的两个子节点是黑色的
      * ⑤对每个节点，从该节点到其所有的后代叶子结点的简单路径上，仅包含相同数目的黑色结点，红黑树是许多“平衡”搜索树的一种，可以保证在最坏情况下的基本操作集合的时间复杂度为O(lgn)。
  * 在TreeSet集合中只能存储相同类型对象的引用。
  * 自然排序和客户化排序



### Map

在java里，哈希表底层是通过数组+链表组成

> Map(映射)是一种键值对映射集合，Map支持多级映射，Map中的键是唯一的，但值可以不唯一，Map集合有两种实现
>

* HashMap：（**==数组+链表+红黑树==、无序、不安全、可以为Null**）
  * 底层通过**数组+链表+红黑树**实现，默认容量为8，在jdk1.8之前是数组+链表，在jdk1.8之后，当位桶的数据超过阈值（8）的时候，就会采用红黑树来存储该位桶的数据
  * 默认大小为  **16**  ，负载因子为0.75（泊松分布），扩容位2的n次幂
  * 插入元素后才判断该不该扩容
  * 计算index方法：index = hash & (tab.length – 1)
  * 哈希冲突 对每个元素执行equals()比较
  * ==当数组大小大于  **64**==，且==链表大于  **8**==  的时候使用红黑树
  * key-value可以为null
  * 线程不安全
  * HashMap，它和HashSet都是利用哈希表来完成的，区别其实就是在哈希表的每个桶中，HashSet只有key，而HashMap在每个key上挂了一个value；



* LinkedHashMap：（**==数组+链表+双向链表==、有序、不安全、可以为空**）
  * 继承与HashMap，在其基础上维护了一个双向链表
  * 通过双向链表实现插入有序
  * 因为遍历的时候用的是双向链表遍历，所以不回影响性能





* HashTable：（**==哈希表==、无序、安全、不可为空**）

  * Hashtable继承Map接口，实现一个key-value映射的哈希表。
  * key-value不可以为null
  * 初始size为**11**，扩容：newsize = olesize*2+1
  * 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length
  * 线程安全 通过（synchronized）




* TreeMap：（**==红黑树==、有序、不安全、值可以为空**）

  * 底层通过**红黑树**（Red-Black tree）实现，实现了SortMap接口，和TreeSet一样也能实现自然排序和客户化排序两种排序方式
  * 继承于AbstractMap，是一个有序的key-value集合，
  * TreeMap 实现了NavigableMap接口，意味着它支持一系列的导航方法。比如返回有序的key集合。
  * TreeMap 实现了Cloneable接口，意味着它能被克隆。
  * TreeMap 实现了java.io.Serializable接口，意味着它支持序列化。
  * TreeMap，它实现了SortMap接口，也就是使用了**红黑树**的数据结构。
  * TreeMap是非同步的。 它的iterator 方法返回的迭代器是fail-fastl的。
  * ==自然排序和客户化排序==



* ConcurrentHashmap：（**数组+链表+红黑树、有序、安全、值可以为空**）JUC包下

  * 通过部分加锁（分段锁）或利用CAS算法实现同步
  * 扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容
  * 1.7使用Segment+HashEntry==分段锁==的方式实现
    * 1.7版本的ConcurrentHashMap采用分段锁机制，里面包含一个Segment数组（Segment继承于ReentrantLock），Segment则包含HashEntry的数组，HashEntry本身就是一个链表的结构，具有保存key、value的能力能指向下一个节点的指针。
  * 实际上就是相当于每个Segment都是一个HashMap，默认的Segment长度是16，也就是支持16个线程的并发写，Segment之间相互不会受到影响。
  * 1.8则抛弃了Segment，改为使用CAS+synchronized+Node实现，同样也加入了红黑树，避免链表过长导致性能的问题。
  
  

### 哈希冲突及四种解决方法

> 哈希是通过对数据进行再压缩 ，提高效率的一种解决方法。但由于通过哈希函数产生的哈希值是有限的，而数据可能比较多，导致经过哈希函数处理后仍然有不同的数据对应相同的值。这时候就产生了哈希冲突。



* 开放地址方法

1. * 线性探测。按顺序决定值时，如果某数据的值已经存在，则在原来值的基础上往后加一个单位，直至不发生哈希冲突。

2. * 再平方探测。按顺序决定值时，如果某数据的值已经存在，则在原来值的基础上先加1的平方个单位，若仍然存在则减1的平方个单位。随之是2的平方，3的平方等等。直至不发生哈希冲突。

3. * 伪随机探测。按顺序决定值时，如果某数据已经存在，通过随机函数随机生成一个数，在原来值的基础上加上随机数，直至不发生哈希冲突。



* 链式地址法：（HashMap的哈希冲突解决方法）对于相同的值，使用链表进行连接。使用数组存储每一个链表。
  * 拉链法处理冲突简单，且无堆积现象，即非同义词决不会发生冲突，因此平均查找长度较短；
  * 由于拉链法中各链表上的结点空间是动态申请的，故它更适合于造表前无法确定表长的情况；
  * 开放定址法为减少冲突，要求装填因子α较小，故当结点规模较大时会浪费很多空间。而拉链法中可取α≥1，且结点较大时，拉链法中增加的指针域可忽略不计，因此节省空间；
  * 在用拉链法构造的散列表中，删除结点的操作易于实现。只要简单地删去链表上相应的结点即可。



* 建立公共溢出区
  * 建立公共溢出区存储所有哈希冲突的数据。



* 再哈希法
  * 对于冲突的哈希值再次进行哈希处理，直至没有哈希冲突。