# 消费者最佳实践续

### 一、消费过程要做到幂等（即消费端去重）

如《RocketMQ 原理简介》中所述，RocketMQ无法避免消息重复，所以如果业务对消费重复非常敏感，务必要在业务层面去重，有以下几种去重方式：

1. 将消息的唯一键，可以是msgId，也可以是消息内容中的唯一标识字段，例如订单Id等，消费之前判断是否在Db或Tair\(全局KV存储\)中存在，如果不存在则插入，并消费，否则跳过。（实际过程要考虑原子性问题，判断是否存在可以尝试插入，如果报主键冲突，则插入失败，直接跳过）；msgId一定是全局唯一标识符，但是可能会存在同样的消息有两个不同msgId的情况（有多种原因），这种情况可能会使业务上重复消费，建议最好使用消息内容中的唯一标识字段去重。
2. 使用业务层面的状态机去重

### 二、消费失败处理方式

### 三、消费速度慢处理方式

**1、  提高消费并行度**

绝大部分消息消费行为属于IO密集型，即可能是操作数据库，或者调用RPC，这类消费行为的消费速度在于后端数据库或者外系统的吞吐量，通过增加消费并行度，可以提高总的消费吞吐量，但是并行度增加到一定程度，反而会下降，如图所示，呈现抛物线形式。所以应用必须要设置合理的并行度。CPU密集型应用除外。

修改消费并行度方法

a\)	同一个ConsumerGroup下，通过增加Consumer实例数量来提高并行度，超过订阅队列数的Consumer实例无效。可以通过加机器，或者在已有机器启动多个进程的方式。

b\)	提高单个Consumer的消费并行线程，通过修改以下参数：`consumeThreadMin`和`consumeThreadMax`。

**2 、批量方式消费**

某些业务流程如果支持批量方式消费，则可以很大程度上提高消费吞吐量，例如订单扣款类应用，一次处理一个订单耗时1秒钟，一次处理10个订单可能也只耗时2秒钟，这样即可大幅度提高消费的吞吐量，通过设置consumer的consumeMessageBatchMaxSize这个参数，默认是1，即一次只消费一条消息，例如设置为N，那么每次消费的消息数小于等于N。

**3、跳过非重要消息**

发生消息堆积时，如果消费速度一直追不上发送速度，可以选择丢弃不重要的消息。如何判断消费发生了堆积？

```java
public ConsumeConcurrentlyStatus consumeMessage(//
        List<MessageExt> msgs, //
        ConsumeConcurrentlyContext context) {
    long offset = msgs.get(0).getQueueOffset();
    String maxOffset = //
         msgs.get(0).getProperty(Message.PROPERTY_MAX_OFFSET);
    long diff = Long.parseLong(maxOffset) - offset;
    if (diff > 100000) {
        // TODO 消息堆积情况的特殊处理
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }

    // TODO 正常消费过程
    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
}
```

如以上代码所示，当某个队列的消息数堆积到100000条以上，则尝试丢弃部分或全部消息，这样就可以快速追上发送消息的速度。

**4、优化每条消息消费过程**

举例如下，某条消息的消费过程如下

1.	根据消息从DB查询数据1

2.	根据消息从DB查询数据2

3.	复杂的业务计算

4.	向DB插入数据3

5.	向DB插入数据4

这条消息的消费过程与DB交互了4次，如果按照每次5ms计算，那么总共耗时20ms，假设业务计算耗时5ms，那么总过耗时25ms，如果能把4次DB交互优化为2次，那么总耗时就可以优化到15ms，也就是说总体性能提高了40%。

对于Mysql等DB，如果部署在磁盘，那么与DB进行交互，如果数据没有命中cache，每次交互的RT会直线上升，如果采用SSD，则RT上升趋势要明显好于磁盘。个别应用可能会遇到这种情况：在线下压测消费过程中，db表现非常好，每次RT都很短，但是上线运行一段时间，RT就会变长，消费吞吐量直线下降。主要原因是线下压测时间过短，线上运行一段时间后，cache命中率下降，那么RT就会增加。建议在线下压测时，要测试足够长时间，尽可能模拟线上环境，压测过程中，数据的分布也很重要，数据不同，可能cache的命中率也会完全不同。

### 四、消费打印日志

如果消息量较少，建议在消费入口方法打印消息，方便后续排查问题。

```java
public ConsumeConcurrentlyStatus consumeMessage(//
            List<MessageExt> msgs, //
            ConsumeConcurrentlyContext context) {
        log.info("RECEIVE_MSG_BEGIN: " + msgs.toString());
        // TODO 正常消费过程
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
}
```

如果能打印每条消息消费耗时，那么在排查消费慢等线上问题时，会更方便。

### 五、利用服务器消息过滤，避免多余的消息传输

无论是通过Tag过滤消息，还是通过MessageFilter过滤。

**RocketMQ通过Tag过滤消息的流程**

\(1\).在Broker端进行Message Tag比对，先遍历Consume Queue，如果存储的Message Tag与订阅的Message Tag不符合，则跳过，继续比对下一个，符合则传输给Consumer。注意：Message Tag是字符串形式，Consume Queue中存储的是其对应的hashcode，比对时也是比对hashcode。

\(2\).Consumer收到过滤后的消息后，同样也要执行在Broker端的操作，但是比对的是真实的Message Tag字符串，而不是Hashcode。

为什么过滤要这样做？

\(1\).	Message Tag存储Hashcode，是为了在Consume Queue定长方式存储，节约空间。

\(2\).	过滤过程中不会访问Commit Log数据，可以保证堆积情况下也能高效过滤。

\(3\).	即使存在Hash冲突，也可以在Consumer端进行修正，保证万无一失。

**RocketMQ通过Filter过滤消息的流程**





内容来源：

[Github vantagewang/document](https://github.com/vintagewang/document) ：RocketMQ最佳实践

[Github vantagewang/document](https://github.com/vintagewang/document) ：RocketMQ原理简介

