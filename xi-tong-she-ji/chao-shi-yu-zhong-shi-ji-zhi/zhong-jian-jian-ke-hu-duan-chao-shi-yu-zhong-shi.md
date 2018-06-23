# 中间件客户端超时与重试

## Dubbo超时与重试

Dubbo是阿里开源的分布式远程调用方案\(RPC\)，由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间。

### Dubbo的超时设置

Provider可以配置的Consumer端主要属性有timeout、retries、loadbalance、actives和cluster。Provider上应尽量多配置些Consumer端的属性，让Provider实现者一开始就思考Provider的服务特点与服务质量。配置之间存在着覆盖，具体规则如下： 

1. 方法级配置别优于接口级别，即小Scope优先 
2. Consumer端配置优于Provider配置，优于全局配置
3. Dubbo Hard Code的配置值（默认）

根据规则2，消费端配置要优于服务端配置，但消费端配置超时时间不能随心所欲，需要根据业务实际情况来设定。如果超时时间设置得太短，复杂业务本来就需要很长时间完成，服务端无法在设定的超时时间内完成业务处理；如果超时时间设置太长，会由于服务端或者网络问题导致客户端资源大量线程挂起。

#### Consumer

```markup
<!--全局超时配置方式-->
<dubbo:consumer timeout="5000" />
<!--指定接口以及特定方法超时配置-->
<dubbo:reference interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:reference>123
```

#### Provider

```markup
<!--全局超时配置-->
<dubbo:provider timeout="5000" />
<!--指定接口以及特定方法超时配置-->
<dubbo:provider interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:provider>
```

### Dubbo的容错机制

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

#### Failover Cluster（默认配置） {#failover-cluster}

失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数\(不含第一次\)。

重试次数配置如下：

```markup
<!--Provider 设置重试-->
<dubbo:service retries="2" />
<!--Consumer 某个接口设置重试-->
<dubbo:reference retries="2" />
<!--Consumer 某个方法设置重试-->
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

#### Failfast Cluster {#failfast-cluster}

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

#### Failsafe Cluster {#failsafe-cluster}

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

#### Failback Cluster {#failback-cluster}

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

#### Forking Cluster {#forking-cluster}

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

#### Broadcast Cluster {#broadcast-cluster}

广播调用所有提供者，逐个调用，任意一台报错则报错（`2.1.0` 开始支持）。通常用于通知所有提供者更新缓存或日志等本地资源信息。

## RocketMQ超时设置

对于消息中间件，一般会较少关注超时问题，因为生产者一般都会默认设置重试次数（当然，这会导致另外一个问题：消息重复，一般需要消费者去重）。

### Producer

#### 超时设置

Producer发送请求给Broker，并等待Broker的响应，但是如果超出最长等待时间Broker还没有返回响应，客户端就会抛出`RemotingTimeoutException`异常。

在Producer端，默认的超时时间是3秒：可以可以通过`sendMsgTimeout`参数设置全局的超时时间，也可以在发送消息时通过通过`send(msg, timeout)`这种方式设置等待时间。 但是**不建议将等待时间设置地很小**，因为Broker需要一些时间来落盘或者将其同步到Slaver Broker。

#### **重试机制**

Producer的`send()`方法本身支持内部重试，重试逻辑如下：

1. 至多重试3次。

2. 如果发送失败，则轮转到下一个Broker。

3. 这个方法的总耗时时间不超过`sendMsgTimeout`设置的值，默认10s。

所以，如果本身向broker发送消息产生超时异常，就不会再做重试（因为已经做过了）。

以上策略仍然不能保证消息一定发送成功，为保证消息一定成功，建议应用这样做：如果调用send同步方法发送失败，则尝试将消息存储到db，由后台线程定时重试，保证消息一定到达Broker。这里有一条最佳实践：**消息发送成功或者失败，要打印消息日志，务必要打印`sendresult`和`key`字段。**

### Consumer

RocketMQ中，Consumer分为PullConsumer和PushConsumer，获取消息的方式分别为拉取和推送。

**对于拉取模式（长轮询）的PullConsumer**而言，可以通过`consumeTimeout`参数设置超时时间，默认15分钟。如果消费超时，RocketMQ会等同于消费失败来处理。如果消息消费失败，则会按照maxReconsumeTimes进行重试，该参数的默认值又根据消费方式分为两种：①并行消费，默认16次，如果16次重试后还是不成功，则消费失败的消息投递到死信队列；②串行消费：默认无限大（Interge.MAX\_VALUE），由于顺序消费的特性必须等待前面的消息成功消费才能消费后面的，默认无限大即一直不断消费直到消费完成。

**对于推送模式的PushConsumer而言**，可以通过如下参数设置超时和重试：

* `brokerSuspendMaxTimeMillis`**：**broker在长轮询下，连接最长挂起的时间，默认20s，RocketMQ不建议修改此值。
* `consumerTimeoutMillisWhenSuspend`**：**broker在长轮询下，客户端等待broker响应的最长等待超时时间，默认30s。RocketMQ不建议修改此值，且此值一定要大于`brokerSuspendMaxTimeMillis`。
* `consumerPullTimeoutMillis`**：**pull的socket 超时时间，虽然注释上说是socket超时时间，但是从源码上看，此值的设计是不启动长轮询也不指定timeout的情况下，拉取的超时时间。
* `maxReconsumeTimes`：调用sendMessageBack的时候，如果发现重新消费超过这个配置的值，则投递到死信队列，默认16。由于PullConsumer没有管理消费的线程池和管理器，需要用户自己处理各种消费结果和拉取结果，故需要投递到重试队列或死信队列的时候需要显示调用`sendMessageBack`。回传消息的时候会带上`maxReconsumeTimes`的值，broker发现此消息已经消费超过此值，则投递到死信队列，否则投递到重试队列。此逻辑和`DefaultPushConsumer`是一致的，只是`PushConsumer`无需用户显示调用。

对Consumer而言，一般有如下最佳实践：①消费过程要做到幂等（即消费端去重）；②解决消费速度慢：  提高消费并行度/批量方式消费/跳过非重要消息/优化每条消息消费过程；③消息量较少，建议在消费入口方法打印消息，方便后续排查问题；④利用服务器消息过滤，避免多余的消息传输。

## HttpClient超时设置

todo







## 参考

[Dubbo超时配置](https://blog.csdn.net/peerless_hero/article/details/68922880)

[Dubbo官方文档：集群容错](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html)

[《RocketMQ用户指南》](https://github.com/vintagewang/document)

RocketMQ官方文档：生产者最佳实践

 [RocketMQ——消息ACK机制及消费进度管理](http://jaskey.github.io/blog/2017/01/25/rocketmq-consume-offset-management/)

[RocketMQ 客户端配置](http://jaskey.github.io/blog/2017/06/13/rocketmq-client-config/)

