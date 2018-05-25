# 部署方式

## Deployment

这一部分介绍生产环境部署方案。即我们要部署一个没有单点故障的集群。

### **Name Server**

为了确保在一个实例崩溃时集群仍然可以运行，建议使用两个或多个Name Server实例，只要集群中有一个实例是可用的，整个集群就可以提供服务。  
Name Server遵从share-nothing设计方式。Brokers会将心跳数据发送到所有Name Server。当要`sending/consuming`时，生产者和消费者可以从任意一台可用的Name server查询到信息。  


### B**roker**

`Brokers`可以按照类别分成两类：`master` 和`slave`。`master`同时提供读写服务，`slave`只提供读服务。  
要部署一个没有单点故障的高可用集群，需要部署多个Broker。一个`broker`集群需要有一个`brokerId`为0的`master`和多个`brokerId`不为0的`slave`。这个`broker`集群的主从需要配置相同的`brokerName`。极端情况下，我们需要保证一个`broker`集群中至少部署两个`broker`服务，每个`topic`都存在于两个或多个`broker`中。  


#### Configuration

当部署一个RocketMQ集群时，我们推荐以下配置：

Broker的推荐配置：

| 属性 | 默认值 | 属性解释 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| listenPort | 10911 | 客户端的连接端口 |
| namesrvAddr | null | name server的地址 |
| brokerIP1 | InetAddress for network interface | 如果有多个地址，就需要配置多个 |
| brokerName | null | 该broker的名字 |
| brokerClusterName | DefaultCluster | 该broker所属集群的名字 |
| brokerId | 0 | broker的 id，0为mater，其他正整数为slave |
| storePathCommitLog | $HOME/store/commitlog/ | commit log的文件路径 |
| storePathConsumerQueue | $HOME/store/consumequeue/ | consume queue的文件路径 |
| mapedFileSizeCommitLog | 1024 \* 1024 \* 1024\(1G\) | mapped file size for commit log |
| deleteWhen | 04 | 超出预定时间后多久删除提交日志 |
| fileReserverdTime | 72 | 提交日志的保留时间，单位小时 |
| brokerRole | ASYNC\_MASTER | SYNC\_MASTER/ASYNC\_MASTER/SLVAE |
| flushDiskType | ASYNC\_FLUSH | 有两种模式可选：SYNC\_FLUSH和ASYNC\_PLUSHBroker；同步模式会在每次消息确认前落盘；异步模式会充分利用分组提交的优势，从而性能更高； |

#### 命令行管理工具（CLI Admin Tool）

RocketMQ提供了一个管理工具，用于查询管理诊断各种问题。该工具是和RocketMQ绑定在一起的，无论你是下载的是编译好的版本还是自己编译的，你的环境中都已经有了（如果你需要源码，那你可以在rocketmq-tools的模块中找到）。

使用管理工具非常简单。为了演示，我们假设你已经在\*nix环境中切换到`${PACKAGE}/bin`目录，输入 `bash mqadmin`，你就能看到如下的帮助菜单。

```text
The most commonly used mqadmin commands are:
   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker
   updateBrokerConfig   Update broker's config
   updateTopicPerm      Update topic perm
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   topicClusterList     get cluster info for topic
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByUniqueKey  Query Message by Unique key
   queryMsgByOffset     Query Message by offset
   queryMsgByUniqueKey  Query Message by Unique key
   printMsg             Print Message Detail
   sendMsgStatus        Send msg to broker
   brokerConsumeStats   Fetch broker consume stats data
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     Clone offset from other group
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config
   deleteKvConfig       Delete KV config
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart)
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   cleanUnusedTopic     Clean unused topic on broker
   startMonitoring      Start Monitoring
   statsAll             Topic and Consumer tps stats
   syncDocs             Synchronize wiki and issue to github.com
   allocateMQ           Allocate MQ
   checkMsgSendRT       Check message send response time
   clusterRT            List All clusters Message Send RT
```

有关特定命令的信息（如`clusterList`），你只需要输入`bash mqadmin help clusterList`：

```bash
usage: mqadmin clusterList [-h] [-i <arg>] [-m] [-n <arg>]
 -h,--help                Print help
 -i,--interval <arg>      specify intervals numbers, it is in seconds
 -m,--moreStats           Print more stats
 -n,--namesrvAddr <arg>   Name server address list, eg: 192.168.0.1:9876;192.168.0.2:9876
```

### 复制模式

为了确保不会丢失发布成功的消息，RocketMQ提供同步和异步两种复制方式来增强消息的可靠性与高可用性：同步复制模式下，master broker会等待提交日志复制到slave broker之后再去确认消息；异步复制模式会在mater broker 处理成功后马上进行消息确认。

**如何配置：**在conf文件夹下，RocketMQ提供了三种默认配置作为参考

```text
2m-2s-sync  //2个Master + 2个Slave，master-slave使用同步复制
2m-2s-async //2个Master + 2个Slave，master-slave使用异步复制
2m-noslave  //2个Master，没有slave
```

注意：所有配置都使用`ASYNC_FLUSH`

**部署**：以2m-2s-sync为例进行部署。

首先启动两个name server，假设他们的ip分别为`192.168.0.2` 和 `192.168.0.3`。

然后启动brokers，假设编译好的RocketMQ在 `/home/rocketmq/dist`目录下，则执行以下命令：

```bash
$ cd /home/rocketmq/dist/bin
$ bash mqbroker -c ../conf/2m-2s-sync/broker-a.properties -n 192.168.0.2:9876,192.168.0.3:9876
$ bash mqbroker -c ../conf/2m-2s-sync/broker-a-s.properties -n 192.168.0.2:9876,192.168.0.3:9876
$ bash mqbroker -c ../conf/2m-2s-sync/broker-b.properties -n 192.168.0.2:9876,192.168.0.3:9876
$ bash mqbroker -c ../conf/2m-2s-sync/broker-b-s.properties -n 192.168.0.2:9876,192.168.0.3:9876
```

如何验证部署成功：使用客户端管理

```bash
$ bash mqadmin clusterlist
```



