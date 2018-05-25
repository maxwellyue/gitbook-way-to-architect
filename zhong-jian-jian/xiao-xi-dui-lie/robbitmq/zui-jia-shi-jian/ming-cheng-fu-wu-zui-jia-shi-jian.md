# 名称服务最佳实践

在Apache RocketMQ中，Name Servers被设计用来分布式系统中的组件间的协同，这些协同主要是通过管理topic路由信息来完成的。 

 ****这些管理工作主要有以下两部分：

* 将Brokers的元数据定期更新到每个Name Server中
* Name Servers将最新的路由信息提供给生产者、消费者以及命令行工具。

因此，在启动Broker和客户端之前，我们需要通过为它们设置Name Server地址来告诉他们如何找到Name Server。在RocketMQ中，有四种方式可以完成这项工作：

### 编码的方式 {#programmatic-way}

对于Brokers而言，我们可以在它的配置文件中定义 `namesrvAddr=name-server-ip1:port;name-server-ip2:port` 。

对生产者和消费者而言， 我们可以通过如下方式告诉它们Name Server的地址列表：

```java
DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
producer.setNamesrvAddr("name-server1-ip:port;name-server2-ip:port");

DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");
consumer.setNamesrvAddr("name-server1-ip:port;name-server2-ip:port");
```

如果你使用命令行工具，你可以这么来定义：

```bash
$ sh mqadmin command-name -n name-server-ip1:port;name-server-ip2:port -X OTHER-OPTION
# 比如查询集群信息
$ sh mqadmin -n localhost:9876 clusterList
```

如果你已经将管理工具整合到你自己的dashboard中，你可以：

```java
DefaultMQAdminExt defaultMQAdminExt = new DefaultMQAdminExt("please_rename_unique_group_name");
defaultMQAdminExt.setNamesrvAddr("name-server1-ip:port;name-server2-ip:port");
```

### Java Options方式 {#java-options}

在启动Java程序时，添加`rocketmq.namesrv.addr`作为Java Options。

### 设置环境变量的方式 {#environment-variable}

你可以将`NAMESRV_ADDR` 设置为环境变量。Broker和客户端如果已经设置了Name Server的地址列表，则会忽略该环境变量。

### HTTP接口的方式 {#http-endpoint}

如果你没有使用上面的三种方式来定义Name Server的地址列表，RocketMQ会访问HTTP接口去获取和更新 Name Server的地址列表。（每两分钟一次，初始化时会延后10秒）。默认的Http接口为：

[`http://jmenv.tbsite.net:8080/rocketmq/nsaddr`](http://jmenv.tbsite.net:8080/rocketmq/nsaddr)

你可以通过`rocketmq.namesrv.domain`这个Java option来重写`jmenv.tbsite.net`。同时也可以通过`rocketmq.namesrv.domain.subgroup`这个Java Option来重写`nsaddr` 。

如果你是在生产环境中使用RocketMQ ，强烈建议使用这种方式。因为这种方式可以给你最大的灵活性：你可以动态地添加或删除Name Server，但不必重启Brokers和客户端。

**以上方式的优先级为：（前面的优先级最高）**  
`Programmatic Way > Java Options > Environment Variable > HTTP Endpoint`

