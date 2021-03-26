#### Netty是什么？

1. Netty 是一个 **基于 NIO** 的 client-server(客户端服务器)框架，使用它可以快速简单地开发网络应用程序。
2. 它极大地简化并优化了 TCP 和 UDP 套接字服务器等网络编程,并且性能以及安全性等很多方面甚至都要更好。
3. **支持多种协议** 如 FTP，SMTP，HTTP 以及各种二进制和基于文本的传统协议。



> ==Netty 成功地找到了一种在不妥协可维护性和性能的情况下实现易于开发，性能，稳定性和灵活性的方法。==

除了上面之外，很多开源项目比如我们常用的 Dubbo、RocketMQ、Elasticsearch、gRPC 等等都用到了 Netty。



#### 为什么要用 Netty？

* 统一的 API，支持多种传输类型，阻塞和非阻塞的。
* 简单而强大的线程模型。
* 自带编解码器解决 TCP 粘包/拆包问题。
* 自带各种协议栈。
* 真正的无连接数据包套接字支持。
* 比直接使用 Java 核心 API 有更高的吞吐量、更低的延迟、更低的资源消耗和更少的内存复制。
* 安全性不错，有完整的 SSL/TLS 以及 StartTLS 支持。
* 社区活跃成熟稳定，经历了大型项目的使用和考验，而且很多开源项目都使用到了 Netty， 比如我们经常接触的 Dubbo、RocketMQ 等等



#### Netty 核心组件有哪些？分别有什么作用？

* Channel
* EventLoop
* ChannelFuture
* ChannelHandler 和 ChannelPipeline