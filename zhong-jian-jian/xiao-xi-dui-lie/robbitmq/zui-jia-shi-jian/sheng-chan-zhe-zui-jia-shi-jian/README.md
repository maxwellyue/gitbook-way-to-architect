# 生产者最佳实践

## 发送状态 SendStatus

生产者发送消息后，会得到`SendResult`，其中就包含`SendStatus`。首先，我们假设消息设置了`isWaitStoreMsgOK=true`（默认值），如果设置为false，则在没有异常抛出的情况下，总会得到SEND\_OK的状态。下面是对每种发送状态的具体解释。（即只要不抛异常，就表示消息已经发送成功了，但是可能并没有完成Slave同步或刷盘）

#### FLUSH\_DISK\_TIMEOUT {#flush_disk_timeout}

如果Broker的消息存储配置了`FlushDiskType=SYNC_FLUSH`\(默认是 `ASYNC_FLUSH`\)，并且Broker没有在配置的落盘时间syncFlushTimeout内（默认是5秒）完成落盘，你将会得到该发送状态。

#### FLUSH\_SLAVE\_TIMEOUT {#flush_slave_timeout}

如果Broke的角色是SYNC\_MASTER（默认是ASYNC\_MASTER），并且Slave Broker在配置的落盘时间syncFlushTimeout内（默认是5秒）没有完成同步Master Broker的工作，你将会得到该发送状态。

#### SLAVE\_NOT\_AVAILABLE {#slave_not_available}

如果Broker的角色是SYNC\_MASTER\(默认是ASYNC\_MASTER\)，但是却没有配置对应的Slave Broke， 你将会得到该发送状态。

#### SEND\_OK {#send_ok}

SEND\_OK这个发送状态并不意味着消息一定是可靠的。为了确保不丢失消息，你应该使用 SYNC\_MASTER 或者SYNC\_FLUSH。

#### Duplication or Missing {#duplication-or-missing}

如果你得到`FLUSH_DISK_TIMEOUT`或者`FLUSH_SLAVE_TIMEOUT`，且此时`Broker`恰巧宕机， 那么你的消息就丢失了。这种情况下，你有两种选择：一个是放任不管，丢了就丢了；二是重发丢失的消息，而重发可能会导致消息重复。通常我们建议重发消息，并在消费消息时解决消费重复问题，除非你觉得消息丢失无所谓。但是请记住，如果你得到`SLAVE_NOT_AVAILABLE`的发送状态，重发是没用的。如果出现`SLAVE_NOT_AVAILABLE`，你应该保持现场并通知Rocket集群管理者来处理。

## 超时问题 Timeout

客户端发送请求给Broker，并等待Broker的响应，但是如果超出最长等待时间Broker还没有返回响应，客户端就会抛出`RemotingTimeoutException`异常。默认的等待时间是3秒。你也可以通过`send(msg, timeout)`这种方式设置等待时间。 注意，我们不建议将等待时间设置地很小，因为Broker需要一些时间来落盘或者将其同步到Slaver Broker。如果等待时间设置的比`syncFlushTimeout`时间还长，可能会这样的效果：在等待超时之前，Broker就可能返回了`FLUSH_SLAVE_TIMEOUT`或者`FLUSH_SLAVE_TIMEOUT`的发送状态（**TODO**：此时，客户端还能获取到Broker返回的这些发送状态吗，还会抛出`RemotingTimeoutException`吗？）。

## 消息大小

建议每个消息的大小不要超过512K。

## 异步发送

默认情况下，发送消息`send(msg)`是同步的（直到收到响应才会结束）。如果你很关心性能，建议使用`(msg, callback)` 这种异步发送消息的方式。

## 生产者组

正常情况下，生产者组has no effects（**TODO**：对什么没有影响？）。 但是如果涉及到事务，你就需要格外注意生产者组。默认情况下，你只需要在同一个JVM中创建一个Producer，一个这就足够了。

## 线程安全

生产者是线程安全的，只需要在你的业务方案中放心使用就行了。

## 性能

如果在大数据处理时，你想在同一个JVM中创建多个Producer，我们建议：

* 使用多个Producer \(3~5 就足够了\)进行异步发送
* 为每个Producer实例设置一个名字。







以上为官方文档翻译。















##  







