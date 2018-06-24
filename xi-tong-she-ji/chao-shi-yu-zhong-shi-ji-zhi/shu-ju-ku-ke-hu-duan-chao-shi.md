# 数据库超时

## 数据库服务端超时设置

以MySQL为例，服务端的超时设置主要有以下几项：（时间单位：秒）

```text
mysql> show global variables like "%timeout%";
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |   # 从5.6.7开始被弃用
| have_statement_timeout      | YES      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 50       |
| innodb_rollback_on_timeout  | OFF      |
| interactive_timeout         | 28800    |
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 60       |
| wait_timeout                | 28800    |
+-----------------------------+----------+
```

#### connect\_timeout

获取MySQL连接是多次握手的结果，除了用户名和密码的匹配校验外，还有IP-&gt;HOST-&gt;DNS-&gt;IP验证，任何一步都可能因为网络问题导致线程阻塞。为了防止线程浪费在不必要的校验等待上，超过connect\_timeout的连接请求将会被拒绝。

**innodb\_lock\_wait\_timeout**  
事务等待获取资源时等待的最长时间，超过这个时间还未分配到资源则会返回应用失败，该参数能够有效避免在资源有限的情况下产生太多的锁等待

**innodb\_rollback\_on\_timeout**

默认情况下`innodb_lock_wait_timeout` 超时后只是超时的sql执行失败，整个事务并不回滚，也不做提交，如需要事务在超时的时候回滚，则需要设置innodb\_rollback\_on\_timeout=ON，该参数默认为OFF。

**interactive\_timeout/wait\_timeout**

即使没有网络问题，也不能允许客户端一直占用连接。对于保持sleep状态超过了wait\_timeout（或interactive\_timeout，取决于client\_interactive标志）的客户端，MySQL会主动断开连接。

**net\_read\_timeout / net\_write\_timeout**

该参数只对TCP/IP链接有效，分别是数据库等待接收客户端发送网络包和发送网络包给客户端的超时时间，这是在Activity状态下的线程才有效的参数。

**slave\_net\_timeout**

Slave判断主机是否挂掉的超时设置，在设定时间内依然没有获取到Master的回应就认为Master挂掉了。

## 数据库客户端超时设置

数据库客户端的超时主要可以分为JDBC超时/连接池超时/Statement超时/事务超时等。这些超时配置的关系和层级如下图所示：

![](../../.gitbook/assets/jdbc-chao-shi-she-zhi.png)

上图中，更上层的超时依赖于下层的超时，只有当较低层的超时机制正常工作，上层的超时才会正常。如果 JDBC 驱动程序的socket超时工作不正常，那么更上层的超时比如 Statement 超时和事务超时都不会正常工作。

### 1、JDBC超时设置

JDBC即数据库的驱动程序，是Java应用中用来连接关系型数据库的标准API。比如我们使用MySQL数据库时，一般会在项目中添加mysql-connector-java.jar，这个库就是JDBC的实现。

JDBC部分主要处理网络连接超时，来处理网络故障 。因为JDBC是属于低级别的库， 高级别的timeout依赖于低级别的timeout，只有当低级别的timeout无误时，高级别的timeout才能确保正常。因此，假如socket timeout出现问题时，高级别的statement timeout和transaction timeout都将失效。 

举个例子，假如JDBC没有设置超时时间，执行某一Statement时设置了超时5s，假如此时发生了网络故障，那么JDBC因为网络故障就会在此挂住，而上层Statement无法感知这种挂起，也会跟着等待，直到JDBC连接成功建立，才开始真正执行该Statement。即Statement时设置的超时5s无效。

### 2、连接池超时设置

数据库连接池，如`DBCP`或最常用的`Driud`，负责的是数据库连接的创建和管理。以`Driud`为例（`DruidDataSource`大部分属性都是参考`DBCP`的），连接池的超时设置有以下几项，主要是**`maxWait`**和**`validationQueryTimeout`**。

* **`maxWait`**：从连接池中获取连接时最大等待时间，单位毫秒。连接池中的连接可能是空闲或正在工作，该参数是指假如连接池中所有连接都在工作，没有空闲的连接可以用，将会等待多久（才有空闲的连接可用）。
* **`validationQueryTimeout`**：单位：秒，检测连接是否有效的超时时间。底层调用`JDBC Statement`对象的`void setQueryTimeout(int seconds)`方法，其实相当于Statement级别的超时设置。 
* `timeBetweenEvictionRunsMillis`：有两个含义： 1\) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于`minEvictableIdleTimeMillis`则关闭物理连接。 2\) `testWhileIdle`的判断依据，`testWhileIdle`是指在申请连接的时候对连接的有效性进行检测，如果空闲时间大于`timeBetweenEvictionRunsMillis`，执行validationQuery检测连接是否有效。
* `minEvictableIdleTimeMillis`：连接保持空闲而不被驱逐的最小时间

  
当在应用中调用`DBCP/Driud`的`getConnection()`方法时，你可以设置获取数据库连接的超时时间，但是这和JDBC的socket超时毫不相关。 ![](http://www.cubrid.org/files/attach/images/220547/584/303/timeout-of-each-level.png)

### 3、Statement超时设置

Statement 超时是用来限制 Statement 的执行时间的，它的具体值是通过下面的JDBC API来设置的。JDBC 驱动程序基于这个值进行 Statement 执行时的超时处理。

```java
// 默认值为0，表示不进行限制
java.sql.Statement.setQueryTimeout(int timeout) 
```

各个JDBC driver实现`java.sql.Statement`的时候都需要实现该函数，超时会直接被取消掉并且抛出`SQLTimeoutException`异常。

但是，一般在开发中，我们一般会使用ORM框架开发，不会出现上面的代码。所以，这个配置更多是通过框架来进行设置（然后再传递给底层的JDBC使用）。以`MyBatis`为例：单位：秒，默认值都是unset，即会使用具体驱动的设置。

```markup
<!-- 在Configuration中设置全局的Statement超时：默认值为unset，即使用具体驱动的设置-->
<configuration>
    ... ...
    <settings>
        ... ...
        <setting name="defaultStatementTimeout" value="25"/>
        ... ...
    </settings>
    
    <typeAliases> ... ...   </typeAliases>    
    <typeHandlers> ... ...  </typeHandlers>
    <plugins>... ... </plugins>
</configuration>

<!-- 在具体的语句中置Statement超时：默认值为unset，即使用具体驱动的设置 -->
<insert id="add" timeout="5">
   ... ... 
</insert>
```

如果使用 Lucy 1.5或1.6版，可以通过设置 `queryTimeout` 属性在数据源层面设置Statement 超时。

可以看出，如果**不设置Statement的超时，则默认不进行超时限制**，这是很危险的做法，因此在开发中，务必在ORM框架中（如果不使用MyBatis之类的ORM框架，则需要在相应的数据库操作工具中）设置Statement的超时。而Statement 超时的具体数值需要根据每个应用自身的情况而定，并没有推荐的配置。

### 4、Transition超时设置

事务超时是在框架（Spring或EJB容器）或应用程序层面上才有效的超时。主要用来限制执行一个事务内所有 Statement 执行的总时长。因此，事务超时可以理解为：

```text
事务超时 = Statement超时时间 * 执行的 Statement 的数量 + 其他垃圾回收等时间
```

比如，假设执行一次Statement 需要0.1秒，那执行几次 Statement并不是什么问题，但如果是执行十万次则需要一万秒（大约7个小时），这就可以用上事务超时了。

以Spring为例： 单位：秒，默认-1，即使用底层事务控制系统的配置，比如在JTA中，默认是30s。 

XML文件配置方式

```markup
<!--全局配置-->
<bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
          p:dataSource-ref="dataSource" 
          p:defaultTimeout="100"/>
<tx:attributes>
        <!--为某个方法单独配置-->
        <tx:method name="…" timeout="100"/>
</tx:attributes>
```

Transactional 注解方式

```java
@Transactional(timeout=100)
public void add(){
    ... ...
}
```

Spring 提供的事务超时的配置非常简单，它会记录每个事务的开始时间和消耗时间，当特定的事件发生时会对已消耗掉的时间做校验，如果超出了配置将抛出异常。

> 在Spring 中，数据库连接会被保存到线程本地变量ThreadLocal中（这被称作事务同步Transaction Synchronization）。当数据库连接被保存到 ThreadLocal 时，同时会记录事务的开始时间和超时时间。所以通过数据库连接的代理创建的 Statement 在执行时就会校验这个时间。



### **FAQ** 

  
Q1. 我已经使用Statement.setQueryTimeout\(\)方法设置了查询超时，但在网络出错时并没有产生作用。 

➔ 查询超时仅在socket timeout生效的前提下才有效，它并不能用来解决外部的网络错误，要解决这种问题，必须设置JDBC的socket timeout。 

Q2. transaction timeout，statement timeout和socket timeout和DBCP的配置有什么关系？ 

➔ 当通过DBCP获取数据库连接时，除了DBCP获取连接时的waitTimeout配置以外，其他配置对JDBC没有什么影响。 

Q3. 如果设置了JDBC的socket timeout，那DBCP连接池中处于IDLE状态的连接是否也会在达到超时时间后被关闭？ 

➔ 不会。socket的设置只会在产生数据读写时生效，而不会对DBCP中的IDLE连接产生影响。当DBCP中发生新连接创建，老的IDLE连接被移除，或是连接有效性校验的时候，socket设置会对其产生一定的影响，但除非发生网络问题，否则影响很小。 

Q4. socket timeout应该设置为多少？ 

➔ socket timeout必须高于statement timeout，但并没有什么推荐值。在发生网络错误的时候，socket timeout将会生效，但是再小心的配置也无法避免网络错误的发生，只是在网络错误发生后缩短服务失效的时间（如果网络恢复正常的话）。

## 参考

[如何配置MySQL数据库超时设置](https://blog.csdn.net/qq_34531925/article/details/78812841)

[MySQL的timeout那点事](http://www.penglixun.com/tech/database/mysql_timeout.html)

[MySQL 各种超时参数的含义](https://www.cnblogs.com/xiaoboluo768/p/6222862.html)

[深入理解JDBC的超时设置](http://www.importnew.com/2466.html)

[Understanding JDBC Internals & Timeout Configuration](https://www.cubrid.org/blog/understanding-jdbc-internals-and-timeout-configuration)

[MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)

[Apache Commons DBCP](http://commons.apache.org/dbcp/)

[DruidDataSource配置属性列表](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

[Spring Transaction Management](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html)

[处理sql超时](http://xiaobaoqiu.github.io/blog/2015/08/22/chu-li-sqlchao-shi/)

[MySQL Configuration Properties for Connector](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html)

[张开涛：超时与重试机制\(2\)](http://zhuanlan.51cto.com/art/201707/543853.htm)

