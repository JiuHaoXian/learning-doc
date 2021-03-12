# Redis

> REmote DIctionary Server(Redis) 是一个由 Salvatore Sanfilippo 写的 key-value 存储系统，是跨平台的非关系型数据库。
>
> Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存、分布式、可选持久性的键值对(Key-Value)存储数据库，并提供多种语言的 API。
>
> Redis 通常被称为数据结构服务器，因为值（value）可以是字符串(String)、哈希(Hash)、列表(list)、集合(sets)和有序集合(sorted sets)等类型。



## 为什要用redis

#### redis和mysql的IO区别

##### mysql

B+Tree在内存中只保存了树干，通过树干找到索引区间（datapage）放入内存，在内存中用二分查找锁定指针，按照同样步骤直至找到做后一个datapage。因为mysql属于行级存储，所以会通过索引去找到的是一整行数据data page。

这里的datapage是磁盘页

**行级存储（Row-based ）：**在创建表时，必须给出**schema**字节宽度(包括列的数据类型，列数量等等）

###### 在磁盘中，一次完整的IO操作为

​		旋转延迟（磁盘轴旋转时间）+寻道时间（磁盘臂移动时间）+带宽（数据传输时间）

磁盘：1. 寻址（毫秒ms），2. 带宽（M）



##### Redis 

当拿到一个key后， redis 先判断当前库的[0]号哈希表是否为空，为空直接返回NULL；判断该0号哈希表是否在rehash，如果正在进行rehash，将调用一次_dictRehashStep方法进行被动 rehash，然后根据计算出的索引值在哈希表中取出链表，遍历该链表找到key的位置，一般情况，该链表长度为1；当 ht[0] 查找完了之后，再进行了次rehash判断，如果未在rehashing，则直接结束，否则对ht[1]重复上述步骤。

内存：1. 寻址（纳秒ns）， 2. 带宽（G）

> 总结：当并发大时，数据库因磁盘寻址与行级存储数据冗余从而占用IO的影响，性能会受很大的影响。



## 数据类型

![image-20210313035720903](assets/image-20210313035720903.png)

## Redis单线程与内核交互原理

#### 单线程如何变得很快的

若干客户端连接（client）先到达内核，Redis通过调用系统内核提供的**epoll**（非阻塞多路复用IO）来处理客户端操作请求。

![image-20210313055110031](assets/image-20210313055110031.png)



