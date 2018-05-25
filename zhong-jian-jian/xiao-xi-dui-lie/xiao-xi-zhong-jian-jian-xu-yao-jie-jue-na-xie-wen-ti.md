# 消息中间件需要解决的问题

## 1、Publish/Subscribe

发布订阅是消息中间件的最基本功能，也是相对于传统RPC通信而言。在此不再详述。

## 2、Message Priority

规范中描述的优先级是指在一个消息队列中，每条消息都有不同的优先级，一般用整数来描述，优先级高的消息先投递，如果消息完全在一个内存队列中，那么在投递前可以按照优先级排序，令优先级高的先投递。

对于优先级问题，可以归纳为2类

1\)	只要达到优先级目的即可，不是严格意义上的优先级，通常将优先级划分为高、中、低，或者再多几个级别。每个优先级可以用不同的topic表示，发消息时，指定不同的topic来表示优先级，这种方式可以解决绝大部分的优先级问题，但是对业务的优先级精确性做了妥协。

2\)	严格的优先级，优先级用整数表示，例如0 ~ 65535，这种优先级问题一般使用不同topic解决就非常不合适。如果要让MQ解决此问题，会对MQ的性能造成非常大的影响。这里要确保一点，业务上是否确实需要这种严格的优先级，如果将优先级压缩成几个，对业务的影响有多大？

## 3、Message Order

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了3条消息，分别是订单创建，订单付款，订单完成。消费时，要按照这个顺序消费才能有意义。但是同时订单之间是可以并行消费的。

## 4、Message Filter

**Broker端消息过滤**

在Broker中，按照Consumer的要求做过滤，优点是减少了对于Consumer无用消息的网络传输。

缺点是增加了Broker的负担，实现相对复杂。

**Consumer端消息过滤**

这种过滤方式可由应用完全自定义实现，但是缺点是很多无用的消息要传输到Consumer端。

## 5、Message Persistence

几种持久化方式：

\(1\).	持久化到数据库，例如Mysql。

\(2\).	持久化到KV存储，例如levelDB、伯克利DB等KV存储系统。

\(3\).	文件记录形式持久化，例如Kafka，RocketMQ

\(4\).	对内存数据做一个持久化镜像，例如beanstalkd，VisiNotify

\(1\)、\(2\)、\(3\)三种持久化方式都具有将内存队列Buffer进行扩展的能力，\(4\)只是一个内存的镜像，作用是当Broker挂掉重启后仍然能将之前内存的数据恢复出来。

JMS与CORBA Notification规范没有明确说明如何持久化，但是持久化部分的性能直接决定了整个消息中间件的性能。

## 6、Message Reliablity 

影响消息可靠性的几种情况：

\(1\).	Broker正常关闭

\(2\).	Broker异常Crash

\(3\).	OS Crash

\(4\).	机器掉电，但是能立即恢复供电情况。

\(5\).	机器无法开机（可能是cpu、主板、内存等关键设备损坏）

\(6\).	磁盘设备损坏。

\(1\)、\(2\)、\(3\)、\(4\)四种情况都属于硬件资源可立即恢复情况，RocketMQ在这四种情况下能保证消息不丢，或者丢失少量数据（依赖刷盘方式是同步还是异步）。

\(5\)、\(6\)属于单点故障，且无法恢复，一旦发生，在此单点上的消息全部丢失。RocketMQ在这两种情况下，通过异步复制，可保证99%的消息不丢，但是仍然会有极少量的消息可能丢失。通过同步双写技术可以完全避免单点，同步双写势必会影响性能，适合对消息可靠性要求极高的场合，例如与Money相关的应用。

## 7、Low Latency Messaging

在消息不堆积情况下，消息到达Broker后，能立刻到达Consumer。

## 8、At least Once

是指每个消息必须投递一次

## 9、Exactly Only Once

\(1\).	发送消息阶段，不允许发送重复的消息。

\(2\).	消费消息阶段，不允许消费重复的消息。

此问题的本质原因是网络调用存在不确定性，即既不成功也不失败的第三种状态，所以才产生了消息重复性问题。

## 10、Broker的Buffer满了怎么办?

Broker 的 Buffer 通常指的是 Broker 中一个队列的内存 Buffer 大小，这类 Buffer 通常大小有限，如果 Buffer 满 了以后怎么办?

下面是 CORBA Notification 规范中处理方式: 

\(1\). RejectNewEvents

拒绝新来的消息，向 Producer 返回 RejectNewEvents 错误码。 

\(2\). 按照特定策略丢弃已有消息

a\) AnyOrder - Any event may be discarded on overflow. This is the default setting for this property.

b\) FifoOrder - The first event received will be the first discarded.

c\) LifoOrder - The last event received will be the first discarded.

d\) PriorityOrder - Events should be discarded in priority order, such that lower priority

events will be discarded before higher priority events.

e\) DeadlineOrder - Events should be discarded in the order of shortest expiry deadline first.



## 11 、回溯消费

回溯消费是指 Consumer 已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，Broker 在向Consumer 投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度，例如由于 Consumer 系统故障，恢复后需要重新消费 1 小时前的数据，那么 Broker 要提供一种机制，可以按照时间维度来回退消费进度。 

## 12、 消息堆积

消息中间件的主要功能是异步解耦，还有个重要功能是挡住前端的数据洪峰，保证后端系统的稳定性，这就要 求消息中间件具有一定的消息堆积能力，消息堆积分以下两种情况:

\(1\). 消息堆积在内存 Buffer，一旦超过内存 Buffer，可以根据一定的丢弃策略来丢弃消息，如 CORBA Notification 规范中描述。适合能容忍丢弃消息的业务，这种情况消息的堆积能力主要在于内存 Buffer 大小，而且消息 堆积后，性能下降不会太大，因为内存中数据多少对于对外提供的访问能力影响有限。

\(2\). 消息堆积到持久化存储系统中，例如DB，KV存储，文件记录形式。

当消息不能在内存 Cache 命中时，要不可避免的访问磁盘，会产生大量读 IO，读 IO 的吞吐量直接决定了 消息堆积后的访问能力。

评估消息堆积能力主要有以下四点:

\(1\). 消息能堆积多少条，多少字节?即消息的堆积容量。

\(2\). 消息堆积后，发消息的吞吐量大小，是否会受堆积影响? \(3\). 消息堆积后，正常消费的Consumer是否会受影响?

\(4\). 消息堆积后，访问堆积在磁盘的消息时，吞吐量有多大?

## 13、分布式事务

已知的几个分布式事务规范，如 XA，JTA 等。其中 XA 规范被各大数据库厂商广泛支持，如 Oracle，Mysql 等。 其中 XA 的 TM 实现佼佼者如 Oracle Tuxedo，在金融、电信等领域被广泛应用。

分布式事务涉及到两阶段提交问题，在数据存储方面的方面必然需要 KV 存储的支持，因为第二阶段的提交回 滚需要修改消息状态，一定涉及到根据 Key 去查找 Message 的动作。

## 14 、定时消息

定时消息是指消息发到 Broker 后，不能立刻被 Consumer 消费，要到特定的时间点或者等待特定的时间后才能 被消费。

如果要支持任意的时间精度，在 Broker 层面，必须要做消息排序，如果再涉及到持久化，那么消息排序要不 可避免的产生巨大性能开销。

## 4.15 消息重试

Consumer 消费消息失败后，要提供一种重试机制，令消息再消费一次。Consumer 消费消息失败通常可以认为 有以下几种情况

1. 由于消息本身的原因，例如反序列化失败，消息数据本身无法处理\(例如话费充值，当前消息的手机号被 注销，无法充值\)等。 这种错误通常需要跳过这条消息，再消费其他消息，而这条失败的消息即使立刻重试消费，99%也不成功， 所以最好提供一种定时重试机制，即过 10s 秒后再重试。

2. 由于依赖的下游应用服务不可用，例如 db 连接不可用，外系统网络不可达等。 遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用 sleep 30s，再 消费下一条消息，这样可以减轻 Broker 重试消息的压力。

  



## 内容来源

[Github vantagewang/document](https://github.com/vintagewang/document) ：RocketMQ原理简介







