# Broker部署方案

推荐的几种Broker集群部署方式，这里的Slave不可写，但可读，类似于Mysql主备方式。

## 方案一 单个Master

这种方式风险较大，一旦Broker重启或者宕机时，会导致整个服务不可用，不建议线上环境使用

## 方案二  多Master模式

一个集群无Slave，全是Master，例如2个Master或者3个Master

优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢）。性能最高。

缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到受到影响。

**具体操作：**

1.先启动Name Server，例如机器IP为：192.168.1.1:9876

```bash
$ nohup sh mqnamesrv &
```

2.在机器A，启动第一个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-noslave/broker-a.properties &
```

3.在机器B，启动第二个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-noslave/broker-b.properties &
```

## 方案三 多Master多Slave模式，异步复制

每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟，毫秒级。

优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为Master宕机后，消费者仍然可以从Slave消费，此过程对应用透明。不需要人工干预。性能同多Master模式几乎一样。

缺点：Master宕机，磁盘损坏情况，会丢失少量消息。

**具体操作**

1.先启动Name Server，例如机器IP为：192.168.1.1:9876

```bash
$ nohup sh mqnamesrv &
```

2.在机器A，启动第一个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a.properties &
```

3.在机器B，启动第二个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b.properties &
```

4. 在机器C，启动第一个Slave

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a-s.properties &
```

5.在机器D，启动第二个Slave

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b-s.properties &
```

## 方案四 多Master多Slave模式，同步双写

每个Master配置一个Slave，有多对Master-Slave，HA采用同步双写方式，主备都写成功，向应用返回成功。

优点：数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与数据可用性都非常高

缺点：性能比异步复制模式略低，大约低10%左右，发送单个消息的RT会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能。

1.先启动Name Server，例如机器IP为：192.168.1.1:9876

```bash
$ nohup sh mqnamesrv &
```

2. 在机器A，启动第一个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-a.properties &
```

3. 在机器B，启动第二个Master

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-b.properties &
```

4. 在机器C，启动第一个Slave

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-a-s.properties &
```

5. 在机器D，启动第二个Slave

```bash
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-sync/broker-b-s.properties &
```

以上`Broker`与`Slave`配对是通过指定相同的`brokerName`参数来配对，`Master`的`BrokerId`必须是0，`Slave`的`BrokerId`必须是大于0的数。另外一个`Master`下面可以挂载多个Slave，同一`Master`下的多个`Slave`通过指定不同的`BrokerId`来区分。

`$ROCKETMQ_HOST`指的`RocketMQ`安装目录，需要用户自己设置此环境变量。



## Broker重启对客户端的影响

Broker重启可能会导致正在发往这台机器的的消息发送失败，RocketMQ提供了一种优雅关闭Broker的方法，通过执行以下命令会清除Broker的写权限，过40s后，所有客户端都会更新Broker路由信息，此时再关闭Broker就不会发生发送消息失败的情况，因为所有消息都发往了其他Broker。

```bash
$ sh mqadmin wipeWritePerm -b brokerName -n namesrvAddr
```

## 内容来源

[Github vantagewang/document](https://github.com/vintagewang/document) ：RocketMQ用户指南

