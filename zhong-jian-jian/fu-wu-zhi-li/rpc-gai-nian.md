# RPC概念

### RPC 是什么？

RPC 的全称是 Remote Procedure Call 是一种进程间通信方式。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的，本质上编写的调用代码基本相同。

### RPC 起源

RPC 这个概念术语在上世纪 80 年代由 [Bruce Jay Nelson](https://en.wikipedia.org/wiki/Bruce_Jay_Nelson) 提出。这里我们追溯下当初开发 RPC 的原动机是什么？在 Nelson 的论文 ["Implementing Remote Procedure Calls"](http://birrell.org/andrew/papers/ImplementingRPC.pdf) 中他提到了几点：

> 1. 简单：RPC 概念的语义十分清晰和简单，这样建立分布式计算就更容易。  
> 2. 高效：过程调用看起来十分简单而且高效。  
> 3. 通用：在单机计算中过程往往是不同算法部分间最重要的通信机制。

通俗一点说，就是一般程序员对于本地的过程调用很熟悉，那么我们把 RPC 作成和本地调用完全类似，那么就更容易被接受，使用起来毫无障碍。Nelson 的论文发表于 30 年前，其观点今天看来确实高瞻远瞩，今天我们使用的 RPC 框架基本就是按这个目标来实现的。

## 什么是RPC?

互联网上的机器大都通过[TCP/IP协议](http://en.wikipedia.org/wiki/Internet_protocol_suite)相互访问，但TCP/IP只是往远端发送了一段二进制数据，为了建立服务还有很多问题需要抽象：

* 数据以什么格式传输？不同机器间，网络间可能是不同的字节序，直接传输内存数据显然是不合适的；随着业务变化，数据字段往往要增加或删减，怎么兼容前后不同版本的格式？
* 一个TCP连接可以被多个请求复用以减少开销么？多个请求可以同时发往一个TCP连接么?
* 如何管理和访问很多机器？
* 连接断开时应该干什么？
* 万一server不发送回复怎么办？
* ...

[RPC](http://en.wikipedia.org/wiki/Remote_procedure_call)可以解决这些问题，它把网络交互类比为“client访问server上的函数”：client向server发送request后开始等待，直到server收到、处理、回复client后，client又再度恢复并根据response做出反应。

[![rpc.png](https://github.com/brpc/brpc/raw/master/docs/images/rpc.png)](https://github.com/brpc/brpc/blob/master/docs/images/rpc.png)

我们来看看上面的一些问题是如何解决的：

* 数据需要序列化，[protobuf](https://github.com/google/protobuf)在这方面做的不错。用户填写protobuf::Message类型的request，RPC结束后，从同为protobuf::Message类型的response中取出结果。protobuf有较好的前后兼容性，方便业务调整字段。http广泛使用[json](http://www.json.org/)作为序列化方法。
* 用户无需关心连接如何建立，但可以选择不同的[连接方式](https://github.com/brpc/brpc/blob/master/docs/cn/client.md#%E8%BF%9E%E6%8E%A5%E6%96%B9%E5%BC%8F)：短连接，连接池，单连接。
* 大量机器一般通过名字服务被发现，可基于[DNS](https://en.wikipedia.org/wiki/Domain_Name_System), [ZooKeeper](https://zookeeper.apache.org/), [etcd](https://github.com/coreos/etcd)等实现。在百度内，我们使用BNS \(Baidu Naming Service\)。brpc也提供["list://"和"file://"](https://github.com/brpc/brpc/blob/master/docs/cn/client.md#%E5%90%8D%E5%AD%97%E6%9C%8D%E5%8A%A1)。用户可以指定负载均衡算法，让RPC每次选出一台机器发送请求，包括: round-robin, randomized, [consistent-hashing](https://github.com/brpc/brpc/blob/master/docs/cn/consistent_hashing.md)\(murmurhash3 or md5\)和 [locality-aware](https://github.com/brpc/brpc/blob/master/docs/cn/lalb.md).
* 连接断开时可以重试。
* 如果server没有在给定时间内回复，client会返回超时错误。

## 哪里可以使用RPC?

几乎所有的网络交互。

RPC不是万能的抽象，否则我们也不需要TCP/IP这一层了。但是在我们绝大部分的网络交互中，RPC既能解决问题，又能隔离更底层的网络问题。

对于RPC常见的质疑有：

* 我的数据非常大，用protobuf序列化太慢了。首先这可能是个伪命题，你得用[profiler](https://github.com/brpc/brpc/blob/master/docs/cn/cpu_profiler.md)证明慢了才是真的慢，其次很多协议支持携带二进制数据以绕过序列化。
* 我传输的是流数据，RPC表达不了。事实上brpc中很多协议支持传递流式数据，包括[http中的ProgressiveReader](https://github.com/brpc/brpc/blob/master/docs/cn/http_client.md#%E6%8C%81%E7%BB%AD%E4%B8%8B%E8%BD%BD), h2的streams, [streaming rpc](https://github.com/brpc/brpc/blob/master/docs/cn/streaming_rpc.md), 和专门的流式协议RTMP。
* 我的场景不需要回复。简单推理可知，你的场景中请求可丢可不丢，可处理也可不处理，因为client总是无法感知，你真的确认这是OK的？即使场景真的不需要，我们仍然建议用最小的结构体回复，因为这不大会是瓶颈，并且追查复杂bug时可能是很有价值的线索。





## 参考

[深入浅出 RPC - 浅出篇](https://blog.csdn.net/mindfloating/article/details/39473807)

[brpc 概述](https://github.com/brpc/brpc/blob/master/docs/cn/overview.md)

