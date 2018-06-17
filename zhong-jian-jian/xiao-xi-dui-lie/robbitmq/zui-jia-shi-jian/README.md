# 最佳实践

## 核心概念

![](../../../../.gitbook/assets/image%20%284%29.png)

根据上面的模型，我们可以深入探讨一些关于消息系统设计的话题：

* 并发消费（Consumer Concurrency）
* 消费者热点问题（Consumer Hot Issues）
* 消费者负载均衡（Consumer Load Balance）
* 消息路由（Message Router）
* 多种连接方式（Connection Multiplex）
* 金丝雀部署（Canary Deployment）

## 生产者

生产者将业务系统中产生的消息发送到Broker，RocketMQ提供了多种发送方式：同步（synchronous），异步\(asynchronous\)，单程\(one-way\)。

#### 生产者组 \(Producer Group\)

相同角色的生产者会被组织到一起。如果一个生产者在提交事务后挂掉了，则生产者组中的其他生产者实例都可以连接Broker执行提交或回滚事务。  
**注意**：考虑到生产者有很强的消息发送能力，only one instance is allowed per producer group to avoid unnecessary initialization of producer instances.

## 消费者

消费者负责从Broker拉取消息，并将消息交给应用去处理。从用户的应用视角来看，消费者可以分为两类：拉模式的消费者（PullConsumer）和推模式的消费者\(PushConsumer\)。

**拉模式的消费者（PullConsumer）：**拉模式的消费者会主动从Broker拉取信息，一次拉取一批，用于消费应用进行消费。

**推模式的消费者（PushConsumer）：**推模式的消费者封装了消息的拉取和消费进程和其他的内部工作细节，只提供给终端用户一个在消息到达时的回调接口。

#### 消费者组

与之前提到的生产者组类似，相同角色的的消费者会被组织到一起，称为**消费者组**。  
消费者组概念的引入，使得消息消费时的负载平衡和容错处理变得非常简单。  
**注意**：同一消费者组的消费者实例必须订阅相同的主题。

## Topic

Topic是生产者发送消息和消费者拉取消息时使用的概念。Topic与生产者与消费者之间是松耦合的。特别地，一个Topic可以有0个或1个或者多个生产者向它发送消息。相反，一个生产者可以向多个不同的Topic发送消息。从消费者角度讲，一个主题可以被0个或者1个或者多个消费者组订阅，同样地，一个消费者组可以订阅一个或多个主题，只要这个消费者组保持一致的订阅即可。

## 消息Message

消息是被传递的信息。消息必须有一个主题，可以理解为你写信事邮寄的地址。消息也可以有一个可选的标签以及额外的键值对。例如，在开发阶段为了调式，你可能在发送消息时设置一个业务key，并通过这个key在broker中查看这些消息。

#### **消息队列 message queue**

主题被划分为一个或多个子主题（sub-topics），这就是消息队列。

#### **Tag**

Tag可以理解为另一种sub-topic，为用户筛选消息提供更强的灵活性。在同一业务模块中，为了不同的目的，你可以为这些消息设置相同Topic，但设置不同的Tag。Tag可以让你的代码变得清晰连贯，还可以为RocketMQ的查询系统提供更好的支持。

**代理Broker**

Broker是RockerMQ系统的重要组成部分。它接受并存储来自生产者发送的消息，并处理来自消费者的拉取消息的请求。它同时也负责维护和消息有关的元数据，包括消费者组、消费进度偏移量以及Topoc/Tag的信息。

## 名称服务 NameServer

名称服务负责提供路由信息。生产者/消费者客户端通过查找topics来获取对应的broker列表。

## 消息模型

* Clustering
* Broadcasting

## 有序消息（Message Order）

如果你使用DefaultMQPushConsumer，那么意味着你可以有序地或并发地消费消息。

#### 有序性 Orderly

有序地消费消息，是指消息的消费顺序与生产者为每个消息队列发送的顺序相同。如果你的使用场景要求必是须顺序的，那么要确保你使用的Topic中只有一个message queue。  
**注意**：如果使用顺序消息功能，则最大的消费并发数 is the number of message queues subscribed by the consumer group。

#### 并发性 Concurrency

当并发消费消息时，最大的并发数只受限于每个消费客户端线程池的配置。  
**警告：**并发模式下无法保证消息的有序性。

  
  




  


  


