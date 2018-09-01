# 不同注册中心的比较

常用的注册中心的对比如下：

| Feature | Consul | zookeeper | etcd | euerka |
| :--- | :--- | :--- | :--- | :--- |
| 服务健康检查 | 服务状态，内存，硬盘等 | \(弱\)长连接，keepalive | 连接心跳 | 可配支持 |
| 多数据中心 | 支持 | — | — | — |
| kv存储服务 | 支持 | 支持 | 支持 | — |
| 一致性 | raft | paxos | raft | — |
| cap | ca | cp | cp | ap |
| 使用接口\(多语言能力\) | 支持http和dns | 客户端 | http/grpc | http（sidecar） |
| watch支持 | 全量/支持long polling | 支持 | 支持 long polling | 支持 long polling/大部分增量 |
| 自身监控 | metrics | — | metrics | metrics |
| 安全 | acl /https | acl | https支持（弱） | — |
| spring cloud集成 | 已支持 | 已支持 | 已支持 | 已支持 |

* 服务的健康检查

Euraka 使用时需要显式配置健康检查支持；Zookeeper,Etcd 则在失去了和服务进程的连接情况下任务不健康，而 Consul 相对更为详细点，比如内存是否已使用了90%，文件系统的空间是不是快不足了。

* 多数据中心支持

Consul 通过 WAN 的 Gossip 协议，完成跨数据中心的同步；而且其他的产品则需要额外的开发工作来实现；

* KV 存储服务

除了 Eureka ,其他几款都能够对外支持 k-v 的存储服务，所以后面会讲到这几款产品追求高一致性的重要原因。而提供存储服务，也能够较好的转化为动态配置服务哦。

* 产品设计中 CAP 理论的取舍

Eureka 典型的 AP,作为分布式场景下的服务发现的产品较为合适，服务发现场景的可用性优先级较高，一致性并不是特别致命。其次 CA 类型的场景 Consul,也能提供较高的可用性，并能 k-v store 服务保证一致性。 而Zookeeper,Etcd则是CP类型 牺牲可用性，在服务发现场景并没太大优势；

* 多语言能力与对外提供服务的接入协议

Zookeeper的跨语言支持较弱，其他几款支持 http11 提供接入的可能。Euraka 一般通过 sidecar的方式提供多语言客户端的接入支持。Etcd 还提供了Grpc的支持。 Consul除了标准的Rest服务api,还提供了DNS的支持。

* Watch的支持（客户端观察到服务提供者变化）

Zookeeper 支持服务器端推送变化，Eureka 2.0\(正在开发中\)也计划支持。 Eureka 1,Consul,Etcd则都通过长轮询的方式来实现变化的感知；

* 自身集群的监控

除了 Zookeeper ,其他几款都默认支持 metrics，运维者可以搜集并报警这些度量信息达到监控目的；

* 安全

Consul,Zookeeper 支持ACL，另外 Consul,Etcd 支持安全通道https.





## 参考

[服务注册中心，Eureka与Zookeeper比较](https://my.oschina.net/thinwonton/blog/1622905)​：比较深入，主要是CAP的取舍

[服务发现比较：Consul vs Zookeeper vs Etcd vs Eureka](https://luyiisme.github.io/2017/04/22/spring-cloud-service-discovery-products/?utm_source=tuicool&utm_medium=referral)：文字来源

[Consul vs. ZooKeeper, doozerd, etcd](https://www.consul.io/intro/vs/zookeeper.html)：TODO：Consul官方文档描述，待阅读

### 

