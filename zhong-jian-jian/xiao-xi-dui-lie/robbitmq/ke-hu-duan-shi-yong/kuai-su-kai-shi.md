# 快速开始



## 下载 & 构建 {#download--build-from-release}

点击 [here](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip) 下载 4.2.0 版本的源码，之后执行以下命令：

```bash
  $ unzip rocketmq-all-4.2.0-source-release.zip
  $ cd rocketmq-all-4.2.0/
  $ mvn -Prelease-all -DskipTests clean install -U
  $ cd distribution/target/apache-rocketmq
```

## 启动名称服务（Name Server） {#start-name-server}

```bash
 $ nohup sh bin/mqnamesrv &
 $ tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
```

## 启动Broker {#start-broker}

```bash
  $ nohup sh bin/mqbroker -n localhost:9876 &
  $ tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
```

## 发送 & 接收消息 {#send--receive-messages}

在发送/接收消息之前，我们需要告诉客户端名称服务的地址。RocketMQ 提供了多种配置方式。简单起见，我们使用设置环境变量 `NAMESRV_ADDR`的方式：

```bash
 $ export NAMESRV_ADDR=localhost:9876
 $ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 $ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

## 关闭所有服务 {#shutdown-servers}

```bash
$ sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

$ sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```



