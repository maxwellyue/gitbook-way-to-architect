# 整体架构

## 概述

Apache RocketMQ是一个具有低延迟、高性能和可靠性、万亿级容量同时具备灵活的可伸缩性的分布式消和流处理平台，它由四个部分组成：name servers， brokers，producers 和 consumers。它们所有都可以水平扩展避免单点故障。

![](../../../.gitbook/assets/image%20%288%29.png)

**名称服集群务 NameServer cluster**  
NameServer服务提供了轻量级的服务发现和路由。每个NameServer服务记录完整的路由信息，提供一致的读写服务，支持快速存储扩展。  
  
**代理服务集群 Broker Cluster**  
Broker通过提供轻量级主题（TOPIC）和队列（QUEUE）机制来处理消息存储。它们支持Push和Pull模型，包含容错机制\(2个副本或3个副本\)，提供了极强的峰值处理能力和按照时间顺序存储数以百万记的消息存储能力，此外，Broker提供了灾难恢复、丰富的度量统计和警报机制，这些都是在传统的消息传递系统中所缺乏的。  
  
**生产者集群 Producer Cluster**  
produce支持分布式部署，分布式的produce通过broker集群提供的各种负载均衡策略将消息发送到broker集群中。发送过程支持快速失败，并且是低延迟的。  
  
**消费者集群 Consumer Cluster**  
消费者也支持在推送和者拉取模式下分布式部署，它还支持集群消费和消息广播。提供实时的消息订阅机制，能够满足大多数消费者的需求。

## 名称服务NameServer

NameServer是一个功能齐全的服务器，主要包括两个功能：

①Broker 管理：nameserver 接受来自broker集群的注册信息并提供心跳来检测他们是否可用。

②路由管理：每一个nameserver都持有关于broker集群和队列的全部路由信息，用来向客户端提供查询。

  
我们知道 ，RocketMQ客户端（生产者/消费者）会从nameserver查询队列的路由信息，客户端是如何知道nameserver的地址的呢？有四种方式能够让客户端获取到nameserver的地址：  
①通过编程：`producer.setNamesrvAddr("ip:port")`  
②java 配置项：使用`rocketmq.namesrv.addr`  
③环境变量 ：使用`NAMESRV_ADDR`  
④HTTP 端点  
更多关于nameserver地址发现的详细信息请参考[这里](http://rocketmq.apache.org/rocketmq/four-methods-to-feed-name-server-address-list/)。

## 代理服务 Broker Server

Broker Server负责消息存储、消息传递、消息查询，保证高可用等等。  
如下图所示，Broker Server的重要子模块有：  
①**远程模块**（Remoting Module）：Broker的入口，处理从客户端发起的请求。  
②**客户端管理**\(Cleint Manager\)： 管理各个客户端（生产者/消费者）还有维护消费者主题订阅。  
③**存储服务**\(Store Service\)：提供向磁盘存储或者查询消息的API。  
④**高可用服务**\(HA Service\)：提供主从Broker的数据同步。  
⑤**索引服务**\(Index Service\)：为消息建立索引，以便消息的快速查询。

![](../../../.gitbook/assets/image%20%285%29.png)

## RocketMQ整体网络通信过程描述

Name Server 是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。

Broker 部署相对复杂，Broker 分为 Master 与 Slave，一个 Master 可以对应多个 Slave，但是一个 Slave 只能对应一个 Master，Master 与 Slave 的对应关系通过指定相同的 BrokerName，不同的 BrokerId 来定义，BrokerId 为 0 表示 Master，非 0 表示 Slave。Master 也可以部署多个。每个 Broker 与 Name Server 集群中的所有节 点建立长连接，定时注册 Topic 信息到所有 Name Server。

Producer 与 Name Server 集群中的其中一个节点\(随机选择\)建立长连接，定期从 Name Server 取 Topic 路 由信息，并向提供 Topic 服务的Broker Master 建立长连接，且定时向Broker Master 发送心跳。Producer 完全无状态，可 集群部署。

Consumer 与 Name Server 集群中的其中一个节点\(随机选择\)建立长连接，定期从 Name Server 取 Topic 路由信息，并向提供 Topic 服务的Broker Master、Slave 建立长连接，且定时向 Master、Slave 发送心跳。Consumer既可以从Broker Master 订阅消息，也可以从Broker Slave 订阅消息，订阅规则由 Broker 配置决定。

## 内容来源

[RocketMQ Architecture](https://rocketmq.apache.org/docs/rmq-arc/)

[RocketMQ中文文档（译）](https://blog.csdn.net/chenaima1314/article/details/79202315)

[Github vantagewang/document](https://github.com/vintagewang/document) ：RocketMQ原理简介

