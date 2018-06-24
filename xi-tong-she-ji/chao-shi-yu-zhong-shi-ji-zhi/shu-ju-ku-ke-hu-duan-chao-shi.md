# 数据库超时

数据库服务端超时设置



JDBC超时设置

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

