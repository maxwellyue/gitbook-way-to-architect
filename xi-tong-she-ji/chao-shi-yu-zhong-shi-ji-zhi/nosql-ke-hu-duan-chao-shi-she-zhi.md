# NoSQL客户端超时设置

## Redis

以Jedis客户端为例，通过如下配置分配等待获取连接池连接的超时时间和网络连接/读超时时间：

```java
PoolJedisConnectionFactory connectionFactory = new PoolJedisConnectionFactory();
//获取连接池连接的超时时间，默认-1L，即不进行限制
connectionFactory.setMaxWaitMillis(maxWaitMillis);
//网络连接/读超时时间：默认2000毫秒
connectionFactory.setTimeout(timeoutInMillis);
```

Jedis在建立Socket时通过如下代码设置超时。

```java
this.socket.connect(new InetSocketAddress(this.host, this.port),this. timeout);
this.socket.setSoTimeout(this.timeout);
```

**配置全局默认的Socket连接/读超时**

可以在JVM启动时通过添加如下参数，来配置默认全局的Socket连接/读超时。即如Httpclient、JDBC等，如果没有配置socket超时，则默认会使用该超时。

```text
-Dsun.net.client.defaultConnectTimeout=60000-Dsun.net.client.defaultReadTimeout=60000
```



## MongoDB

MongoDB提供了多个版本的Java客户端驱动。基本选项如下（详见`com.mongodb.MongoOptions`）：

```markup
<beans>
  <mongo:mongo host="localhost" port="27017">
    <mongo:options connections-per-host="8"
                   threads-allowed-to-block-for-connection-multiplier="4"
                   <!---->
                   connect-timeout="1000"
                   max-wait-time="1500"
                   auto-connect-retry="true"
                   socket-keep-alive="true"
                   socket-timeout="1500"
                   slave-ok="true"
                   write-number="1"
                   write-timeout="0"
                   write-fsync="true"/>
  </mongo:mongo/>
</beans>
```

参数说明：（单位均为毫秒）

* `connect-timeout`：新建socket连接时的超时，默认为0，表示不限制
* `socket_timeout`：I/O读写操作时，socket连接的超时，默认为0，表示不限制
* `max-wait-time`：某个线程获取连接时最大等待时间，默认12000，即12s，0表示不限制
* `write-timeout`：写操作的超时时间，默认0，表示不限制



## 参考

[jedis](https://github.com/xetorthio/jedis)

[MongoDB进阶（八）Spring整合MongoDB（Spring Data MongoDB）](https://blog.csdn.net/qq_16313365/article/details/70142729)

[Spring Data MongoDB 文档：配置](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#mongo.mongo-xml-config)

《亿级流量网站架构核心技术》：超时与重试机制





