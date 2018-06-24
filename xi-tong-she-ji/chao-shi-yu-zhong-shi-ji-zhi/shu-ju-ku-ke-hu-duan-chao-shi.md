# 数据库超时

## 数据库服务端超时设置



## JDBC超时设置

JDBC即数据库的驱动程序，是Java应用中用来连接关系型数据库的标准API。比如我们使用MySQL数据库时，一般会在项目中添加mysql-connector-java.jar，这个库就是JDBC的实现。

JDBC通过socket对字节流进行处理，因此会有一些基本网络操作，若没有合理地设置socket timeout，可能会出现连接被阻塞这样的错误。 

JDBC是属于低级别的库， 高级别的timeout依赖于低级别的timeout，只有当低级别的timeout无误时，高级别的timeout才能确保正常。例如，当socket timeout出现问题时，高级别的statement timeout和transaction timeout都将失效。 

即使设置了statement timeout，当网络出错时，应用也无法从错误中恢复。

statement timeout无法处理网络连接失败时的超时，它能做的仅仅是限制statement的操作时间。网络连接失败时的timeout必须交由JDBC来处理。   
JDBC的socket timeout会受到操作系统socket timeout设置的影响，这就解释了为什么在之前的案例中，JDBC连接会在网络出错后阻塞30分钟，然后又奇迹般恢复，即使我们并没有对JDBC的socket timeout进行设置。 

DBCP负责的是数据库连接的创建和管理，并不干涉timeout的处理。当连接在DBCP中创建，或是DBCP发送校验query检查连接有效性的时候，socket timeout将会影响这些过程，但并不直接对应用造成影响。   
当在应用中调用DBCP的getConnection\(\)方法时，你可以设置获取数据库连接的超时时间，但是这和JDBC的timeout毫不相关。 ![](http://www.cubrid.org/files/attach/images/220547/584/303/timeout-of-each-level.png)

连接池超时设置

Statement超时设置

Transition超时设置











## Druid超时设置

DruidDataSource大部分属性都是参考DBCP的，超时相关配置加了粗体显示。

| 配置 | 缺省值 | 说明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| maxActive | 8 | 最大连接池数量 |
| minIdle |  | 最小连接池数量 |
| **maxWait** |  | **获取连接时最大等待时间**，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| validationQuery |  | 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。 |
| **validationQueryTimeout** |  | 单位：秒，**检测连接是否有效的超时时间**。底层调用jdbc Statement对象的void setQueryTimeout\(int seconds\)方法 |
| testOnBorrow | true | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn | false | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testWhileIdle | false | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于`timeBetweenEvictionRunsMillis`，执行validationQuery检测连接是否有效。 |
| keepAlive | false （1.0.28） | 连接池中的`minIdle`数量以内的连接，空闲时间超过`minEvictableIdleTimeMillis`，则会执行keepAlive操作。 |
| **timeBetweenEvictionRunsMillis** | 1分钟（1.0.14） | 有两个含义： 1\) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于`minEvictableIdleTimeMillis`则关闭物理连接。 2\) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| **minEvictableIdleTimeMillis** |  | 连接保持空闲而不被驱逐的最小时间 |





参考

[如何配置MySQL数据库超时设置](https://blog.csdn.net/qq_34531925/article/details/78812841)

[MySQL的timeout那点事](http://www.penglixun.com/tech/database/mysql_timeout.html)

[深入理解JDBC的超时设置](http://www.importnew.com/2466.html)

