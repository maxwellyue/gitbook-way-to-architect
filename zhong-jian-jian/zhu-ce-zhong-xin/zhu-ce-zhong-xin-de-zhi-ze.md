# 注册中心的职责

服务注册中心，给客户端提供可供调用的服务列表，客户端在进行远程服务调用时，根据服务列表然后选择服务提供方的服务地址进行服务调用。服务注册中心在分布式系统中大量应用，是分布式系统中不可或缺的组件，

例如RocketMQ的Name Server，HDFS中的NameNode，Dubbo中的zookeeper注册中心，Spring Cloud中的服务注册中心Eureka。

著名的CAP理论指出，一个分布式系统不可能同时满足C\(一致性\)、A\(可用性\)和P\(分区容错性\)。由于分区容错性在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。



来源：[服务注册中心，Eureka与Zookeeper比较](https://my.oschina.net/thinwonton/blog/1622905)

