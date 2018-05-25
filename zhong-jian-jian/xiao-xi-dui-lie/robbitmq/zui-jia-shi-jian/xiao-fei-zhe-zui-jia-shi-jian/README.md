# 消费者最佳实践

### 消费者组和订阅 Consumer Group and Subscriptions {#consumer-group-and-subscriptions}

你应该注意的第一件事就是：不同的消费者组可以消费相同Topic的消息，且每个消费者组都会有各自的消费偏移量。请务必确保同一组类的消费者订阅了相同的Topics。

## 消息监听器 Message Listener

#### 有序性 {#orderly}

消费者会锁定每一个消息队列（ MessageQueue）来确保消费的顺序性。这会带来性能损耗，但如果你在意消息的顺序，它就很有用。通过不建议抛出异常，你应该返回`ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT`来替代。（**TODO**：哪里抛出异常，这个状态优势干啥的？）

#### 并发性 {#concurrently}

消费者会并发消费消息。通常在追求高性能的场景下才使用该特性。通常不建议抛出异常，你应该返回 `ConsumeConcurrentlyStatus.RECONSUME_LATER` 来代替。（**TODO**：为啥抛出异常，这个状态优势干啥的？）

#### 消费状态 Consumer Status {#consume-status}

For MessageListenerConcurrently，你可以返回`RECONSUME_LATER`来告诉消费者： 现在不能消费，可以稍后再来消费（can not consume it right now and want to reconsume it later）。然后，你可以继续消费其他消息。此时，为了消息的有序性，你又不能跳过这条消息，但你可以返回`SUSPEND_CURRENT_QUEUE_A_MOMENT`告诉消费者让它等一会。（**TODO**：没太懂...）

#### 阻塞 {#blocking}

不建议阻塞消息监听器，因为它会阻塞线程池，且可能导致最终停止消费消息。

### 线程数 {#thread-number}

消费者内部使用线程池来处理消费过程，所以，你可以通过设置`setConsumeThreadMin` 或者 `setConsumeThreadMax`来修改线程池中的线程数量。

### 消费的起点 {#consumefromwhere}

当创建一个新的消费者组的时候，它需要配置是否消费Broker中已经存在的历史消息。. 如果设置为`CONSUME_FROM_LAST_OFFSET`，消费者会忽略历史消息，只消费从它创建后产生的新的消息。 如果设置了`CONSUME_FROM_FIRST_OFFSET`，消费者会消费Broker中已经存在的每一个消息。 你也可以设置`CONSUME_FROM_TIMESTAMP`来让消费者去消费从某一时间点之后产生的消息。

### 消息重复 {#duplication}

很多情况下都会导致消息重复，比如：

* 消费者重发消息\(i.e, `FLUSH_SLAVE_TIMEOUT`的时候\)
* 消费者宕机，导致消费偏移量没有及时更新到Broker。

所有，如果你的应用不能容忍消息重复，你需要做一些额外的工作来处理这个问题，比如，你可以校验你数据库的主键。

