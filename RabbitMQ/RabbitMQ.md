#### 如何保证RabbitMQ不被重复消费？

答：保证消息的唯一性，就算是多次传输，不要让消息的多次消费带来影响，保证消息等幂性；

比如：在写入消息队列的数据做唯一标示，消费消息时，根据唯一标识判断是否消费过

#### 如何保证RabbitMQ消息的顺序性？

答：对消息进行编号，消费者处理消息是根据编号处理消息；



#### 如何保证消息的不丢失？

##### 生产者：

**有两个解决办法：事务机制和confirm机制，最常用的是confirm机制**

RabbitMQ 提供了事务功能，生产者发送数据之前开启 RabbitMQ 事务channel.txSelect，然后发送消息，如果消息没有成功被 RabbitMQ 接收到，那么生产者会收到异常报错，此时就可以回滚事务channel.txRollback，然后重试发送消息；如果收到了消息，那么可以提交事务channel.txCommit。

RabbitMQ可以开启 confirm 模式，==在生产者那里设置开启 confirm 模式之后==，==生产者每次写的消息都会分配一个唯一的 id==，如果消息成功写入 RabbitMQ 中，RabbitMQ 会给生产者==回传一个 ack 消息==，告诉你说这个消息 ok 了。如果 RabbitMQ 没能处理这个消息，会==回调你的一个 nack 接口==，告诉你这个消息接收失败，生产者可以发送。而且你可以结合这个机制自己在内存里维护每个消息 id 的状态，如果超过一定时间还没接收到这个消息的回调，那么可以重发。

注意：RabbitMQ的==事务机制是同步的==，很耗型能，会降低RabbitMQ的吞吐量。==confirm机制是异步的==，生成者发送完一个消息之后，不需要等待RabbitMQ的回调，就可以发送下一个消息，当RabbitMQ成功接收到消息之后会自动异步的回调生产者的一个接口返回成功与否的消息。



##### 队列：

开启RabbitMQ的持久化。当生产者把消息成功写入RabbitMQ之后，RabbitMQ就把消息持久化到磁盘。结合上面的说到的confirm机制，只有==当消息成功持久化==磁盘之后，才会==回调生产者的接口==返回ack消息，

持久化的配置：

第一点是==创建 queue 的时候将其设置为持久化==，这样就可以==保证 RabbitMQ 持久化 queue 的元数据==，但是它是不会持久化 queue 里的数据的。
第二个是发送消息的时候将消息的 ==deliveryMode 设置为 2==，就是将消息设置为持久化，此时 RabbitMQ 就会将消息持久化到磁盘上去。

**注意：持久化要起作用必须同时设置这两个持久化才行，RabbitMQ 哪怕是挂了，再次重启，也会从磁盘上重启恢复 queue，恢复这个 queue 里的数据。**



##### 消费者：

关闭 RabbitMQ 的自动 `ack`消息确认，可以通过一个手动 调用api 来ack



#### RabbitMQ的六种工作模式？

1. ##### simple简单模式

2. ##### work工作模式(资源的竞争)

3. ##### publish/subscribe发布订阅(共享资源)

4. ##### routing路由模式

5. ##### topic 主题模式(路由模式的一种)

6. ##### RPC：远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的思想。

   > RPC 框架的重要组成：

   - 客户端(Client)：服务调用方。
   - 客户端存根(Client Stub)：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。
   - 服务端存根(Server Stub)：接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理。
   - 服务端(Server)：服务的真正提供者。
   - Network Service：底层传输，可以是 TCP 或 HTTP。



#### 哪五种交换机？

* Direct exchange

  * 会根据routingKey 完全匹配成功后才会消费。比如：如果生产一条消息 “我是中国人”，发送到交换机的时候绑定了路由键是：“中国”，则如果要消费的话只有匹配了路由键是“中国”的才能消费。（可以比喻为交换机是 “地球”，路由键是“国家—中国”，消息是“人”，这个消息的身份证是哪个国家的“路由键”那就是只能在这个国家享有权益。）
  * 如果都消费同一个routingKey的话，多个消费者谁先消费到就是谁的

* Topic exchange

  > 应用：订阅任务，信息分类更新业务

  * 该模式不仅仅需要exchange和queue绑定还需要和路由键routingKey关联
  * 模糊匹配模式，比如：两个路由键 animal.dog ， animal.dog.eat。如果该队列不仅仅对“dog”的消息感兴趣，同时还对与“dog”相关的消息感兴趣就可以使用topic模式，animal.dog.# 。（支持# 0或多词模糊匹配，*一个词匹配）

* Fanout exchange

  > 应用：群聊功能、全网消息推送功能

  * 该模式不需要路由键routingKey

  * 该模式只需要将queue和exchange绑定就好。一个exchange可以绑定N多个queue，每一个queue都会得到同样的消息

  * 一个queue可以和多个exchange绑定，消费来自不同的exchange的消息

  * 转发消息最快

    

* Headers exchange：1. 无路由键routingKey的概念。2. 是以 header和message中的消息匹配上才能消费

* System exchange：其实就是系统默认和direct模式没区别，只不过不需要定义exchange名字而已。





#### RabbitMQ 中的 broker 是指什么？cluster 又是指什么？

`broker` 是指一个或多个 `erlang node` 的逻辑分组，且 `node` 上运行着 `RabbitMQ` 应用程序。
`cluster` 是在 `broker` 的基础之上，增加了 `node` 之间共享元数据的约束。



#### 消息基于什么传输？

由于TCP连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ使用信道的方式来传输数据。信道是建立在真实的TCP连接内的虚拟连接，且每条TCP连接上的信道数量没有限制。



####  消息如何分发？

若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。



#### 如何保证RabbitMQ的高可用？

答：没有哪个项目会只用一搭建一台RabbitMQ服务器提供服务，风险太大；



#### 消息队列的作用与使用场景

异步：批量数据异步处理（批量上传文件）
削峰：高负载任务负载均衡（电商秒杀抢购）
解耦：串行任务并行化（退货流程解耦）
广播：基于Pub/Sub实现一对多通信



### 消息幂等性

生产者方面：可以对每条消息生成一个msgID，以控制消息重复投递

```java
 AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
	porperties.messageId(String.valueOF(UUID.randomUUID()))
```

消费者方面：消息体中必须携带一个业务ID，如银行流水号，消费者可以根据业务ID去重，避免重复消费



#### 多个消费者监听一个队列时，消息如何分发?

- 轮询: 默认的策略，消费者轮流，平均地接收消息
- 公平分发: 根据消费者的能力来分发消息，给空闲的消费者发送更多消息



#### 如何保证RabbitMQ消息的可靠传输？

答：消息不可靠的情况可能是**消息丢失**，**劫持**等原因；

丢失又分为：**生产者丢失消息、消息列表丢失消息、消费者丢失消息**

生产者丢失消息：==从生产者弄丢数据这个角度来看，RabbitMQ提供（事务）transaction和（确认）confirm模式来确保生产者不丢消息==；

消息队列丢数据：==消息持久化。==

消费者丢失消息：==消费者丢数据一般是因为采用了自动确认消息模式，改为手动确认消息即可==



