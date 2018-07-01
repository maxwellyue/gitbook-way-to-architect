# 配置中心的实现与选型

## 配置中心实现思路

按照配置中心的功能点划分，其实现思路如下：

#### **配置分发**

实现有两种方式

* 推：实时性变更，需要应用和配置中心保持长连接，复杂度高
* 拉：实时性相对差，没有增量更新机制会增加配置中心压力，复杂度低

#### 订阅和发布

支持配置变更通知，如果是推送，server把每次变更实时发送给订阅客户端  
如果是拉取，则通过比较client和server的数据md5来实现有效变更通知。server会下发数据和md5给client，client请求的时候会带上md5，假如md5变化，则server重新下发，否则无需变更，返回”无变更”

#### 多环镜、集群配置管理

同一份程序在不同的环境（开发，测试，生产）、不同的集群（如不同的数据中心）经常需要有不同的配置，所以需要有完善的环境、集群配置管理

#### 权限控制和操作审计

配置能改变程序的行为，需要有完善的权限控制，以防错误的配置引起的故障。变更操作都要有审计日志，可以方便的追踪问题

#### 版本管理

所有的配置发布都有版本概念，从而可以方便的支持配置的回滚

#### 支持灰度发布

支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例，有两种解决方案：

* 配置项增加一个host属性，表示这个配置项只“发布”给某些IP
* 定义一个优先级，客户端优先加载本地配置文件，这样如果某些机器上的应用需要特殊配置，那么可以采用老的方式上去修改其本地配置文件

#### 支持降级

非核心服务降级开关。发生问题时，降级服务的非核心服务。

#### 其他

* 同时支持web页面和Restful API接口
* 支持多语言，支持php这种无状态语言
* 容错，当server出现问题时，不影响client的运行
* 支持本地缓存，减少每次获取配置项时与server的网络消耗，仅server更新时通知client

**客户端的支持**

* 配合配置中心的推送，在参数变化时调用客户端自行实现的回调接口，不需要重启应用
* 支持环境变量，JVM启动参数，配置文件，配置中心等多种来源按优先级互相覆盖，并有接口暴露最后的参数选择
* 配置文件中支持多套profile，如开发，单元测试，集成测试，生产

## **开源解决方案现状**

#### Diamond

阿里巴巴中间件部门很早就自研了配置中心Diamond，并且是开源的。Diamond对阿里系统的灵活稳定性发挥了至关重要的作用。开源版本的Diamond由于研发时间比较早，使用的技术比较老，功能也不够完善，目前社区不热已经不维护了。

[Apollo](https://github.com/ctripcorp/apollo)

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

目前提供了以下的特性：

* **统一管理不同环境、不同集群的配置**
* **配置修改实时生效：**用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序
* **版本发布管理：**所有的配置发布都有版本概念，从而可以方便地支持配置的回滚
* **灰度发布：**支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例
* **权限管理、发布审核、操作审计**
* **客户端配置信息监控，**可以在界面上方便地看到配置在被哪些实例使用
* **提供Java和.Net原生客户端，**非Java和.Net应用使用HTTP接口。
* **提供开放平台API**
* **部署简单：**目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来。

#### [disconf](https://github.com/knightliao/disconf/tree/master/disconf-web)

百度开源的配置中心产品，作者是前百度资深工程师廖绮绮。在Apollo没有出来之前，Disconf在社区是比较火的，但是自从廖琦琦离开百度之后，他好像没有足够精力投入维护这个项目，目前社区活跃度已经大不如前，只支持Java＋Spring。

#### [Qconf](https://github.com/Qihoo360/QConf)

QConf是360开源的一个分布式配置管理工具。基于zookeeper实现，特色是基于Agent模式的多语言支持（c/c++、shell、php、python、lua、java、go、node等），但服务端也没有界面、灰度、预案什么的，直接通过API操作ZK而已。

#### Gatekeeper

Facebook内部也有一整套完善的配置管理体系，其中一个产品叫Gatekeeper，目前没有开源。

#### 其他

[cfg4j](http://www.cfg4j.org/)、[xdiamond](https://github.com/hengyunabc/xdiamond) 、[zk-ucc](https://github.com/cncduLee/zk-ucc)、[xxlconf](https://github.com/xuxueli/xxl-conf)、[diablo](https://github.com/ihaolin/diablo)、[super-diamond](https://github.com/melin/super-diamond)

## 其他开源组件

可以基于现有的一些开源组件，实现配置中心的功能。

TODO

[etcd](https://github.com/coreos/etcd)

[zookeeper](https://zookeeper.apache.org/)

[consul](https://www.consul.io/)

[confd](https://github.com/kelseyhightower/confd)

confd是用go写的一个轻量级配置管理工具，它主要关注

* 使用存储在etcd/consul/dynamodb/redis/vault/zookeeper/aws ssm参数存储/环境变量中的配置，并处理模板资源中的数据，使本地配置文件保持最新。
* 重新加载应用程序以获取新的配置文件更改













## 内容来源

[配置中心那点事](http://deadline.top/2016/11/23/%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E9%82%A3%E7%82%B9%E4%BA%8B/)

[开源分布式配置中心选型  
](http://vernonzheng.com/2015/02/09/%E5%BC%80%E6%BA%90%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E9%80%89%E5%9E%8B/)

