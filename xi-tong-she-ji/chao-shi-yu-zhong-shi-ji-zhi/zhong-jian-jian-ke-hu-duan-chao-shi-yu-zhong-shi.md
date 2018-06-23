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







## HttpClient超时设置







## 参考

[Dubbo超时配置](https://blog.csdn.net/peerless_hero/article/details/68922880)

[Dubbo官方文档：集群容错](https://dubbo.gitbooks.io/dubbo-user-book/content/demos/fault-tolerent-strategy.html)



