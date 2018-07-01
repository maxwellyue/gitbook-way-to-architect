# Consul是什么

[Consul](https://github.com/hashicorp/consul)是[HashiCorp](https://www.hashicorp.com/)公司公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其他分布式服务注册与发现的方案，比如 [Airbnb](https://www.airbnb.com/)的[SmartStack](http://nerds.airbnb.com%20/smartstack-service-discovery-cloud/)等相比，Consul的方案更“一站式”（不再需要依赖其他工具，如[ZooKeeper](http://tonybai.com/tag/zookeeper)等）：

* 服务注册与发现
* 健康检查
* Key/Value存储
* 多数据中心方案

使用起来也较为简单。Consul用[Golang](http://tonybai.com/tag/go)实现，因此具有天然可移植性\(支持Linux、windows和Mac OS X\)；安装包仅包含一个可执行文件，方便部署，与[Docker](http://tonybai.com/tag/docker)等轻量级容器可无缝配合。

## Consul Cluster

在Consul方案中，每个提供服务的节点上都要部署和运行Consul的agent，所有运行Consul agent节点的集合构成Consul Cluster。Consul agent有下述两种运行模式。

#### Server 

以Server模式运行的Consul agent称为server，用于维护Consul集群的状态。官方建议每个Consul Cluster至少有3个或以上的Server mode Agent。

#### Client

以Client模式运行的Consul agent称为client，无状态，仅仅负责将请求转发给Server agent节点。

Consul的整体结构如下图所示：  


![](../../.gitbook/assets/image%20%2811%29.png)

每个数据中心的`Consul Cluster`都会在运行于server模式下的agent节点中选出一个Leader节点，这个选举过程通过Consul实现的raft协议保证，多个 server节点上的Consul数据信息是强一致的。  




内容来源

[使用consul实现分布式服务注册和发现](https://tonybai.com/2015/07/06/implement-distributed-services-registery-and-discovery-by-consul/)

