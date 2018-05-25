---
description: 对用户而言，一些很有用的技巧
---

# Broker的最佳实践

## Broker的角色

Broker的角色有：ASYNC\_MASTER，SYNC\_MASTER，SLAVE。

如果不能容忍消息丢失，请使用SYNC\_MASTER，并为该Master设置一个Slaver。

如果可以容忍消息丢失，但是想让Broker一直可用，你应该部署ASYNC\_MASTER，并为该Master设置一个Slaver。

如果仅仅是想尽可能简单，你只需要部署一个ASYNC\_MASTER即可，且无需为它设置Slave。

## 落盘类型 FlushDiskType

推荐使用ASYNC\_FLUSH模式，因为SYNC\_FLUSH模式expensive，且性能消耗很大。如果你觉得ASYNC\_FLUSH模式不够稳定或安全，你可以使用SYNC\_MASTER加一个SLAVE的方式。

## ReentrantLock vs CAS

to be finished

## os.sh

to be finished



翻译自官方文档：[Best Practice For Broker](https://rocketmq.apache.org/docs/best-practice-broker/)

