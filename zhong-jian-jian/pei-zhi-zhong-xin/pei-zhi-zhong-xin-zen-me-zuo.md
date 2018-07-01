# 配置中心的实现与选型

**配置中心怎么做**

* 配置分发，实现有两种方式 推：实时性变更，需要应用和配置中心保持长连接，复杂度高 拉：实时性相对差，没有增量更新机制会增加配置中心压力，复杂度低
* 订阅和发布 支持配置变更通知，如果是推送，server把每次变更实时发送给订阅客户端 如果是拉取，则通过比较client和server的数据md5来实现有效变更通知。server会下发数据和md5给client,client请求的时候会带上md5,假如md5变化，则server重新下发，否则无需变更，返回”无变更”
* 多环镜、集群配置管理 同一份程序在不同的环境（开发，测试，生产）、不同的集群（如不同的数据中心）经常需要有不同的配置，所以需要有完善的环境、集群配置管理
* 权限控制和操作审计：配置能改变程序的行为,需要有完善的权限控制，以防错误的配置引起的故障。变更操作都要有审计日志，可以方便的追踪问题
* 版本管理 所有的配置发布都有版本概念，从而可以方便的支持配置的回滚
* 支持灰度发布 支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例,两种解决方案： a.配置项增加一个host属性，表示这个配置项只“发布”给某些IP b.定义一个优先级，客户端优先加载本地配置文件，这样如果某些机器上的应用需要特殊配置，那么可以采用老的方式上去修改其本地配置文件
* 支持降级 非核心服务降级开关。发生问题时，降级服务的非核心服务
* 同时支持web页面和Restful API接口
* 支持多语言，支持php这种无状态语言
* 支持容错性，当server出现问题时，不影响client的运行
* 支持本地缓存，减少每次获取配置项时与server的网络消耗，仅server更新时通知client

**客户端怎么支持**

* 配合配置中心的推送，在参数变化时调用客户端自行实现的回调接口，不需要重启应用
* 支持环境变量，JVM启动参数，配置文件，配置中心等多种来源按优先级互相覆盖，并有接口暴露最后的参数选择
* 配置文件中支持多套profile，如开发，单元测试，集成测试，生产

**开源解决方案现状**

* 淘宝的[Diamond](http://deadline.top/2016/11/23/%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E9%82%A3%E7%82%B9%E4%BA%8B/)
* 携程开源的[Applo](https://github.com/ctripcorp/apollo)，支持Java，其他语言通过Http支持
* 个人开源的[disconf](https://github.com/knightliao/disconf/tree/master/disconf-web)，只支持Java＋Spring
* 360的[Qconf](https://github.com/Qihoo360/QConf)，基于zk，特色是基于Agent模式的多语言支持。但服务端也没有界面、灰度、预案什么的，直接通过API操作ZK而已
* 个人开源的[xdiamond](https://github.com/hengyunabc/xdiamond)
* 个人开源的[zk-ucc](https://github.com/cncduLee/zk-ucc)
* 个人开源的[xxlconf](https://github.com/xuxueli/xxl-conf)
* 个人开源的[diablo](https://github.com/ihaolin/diablo)
* [etcd](https://github.com/coreos/etcd)
* [zookeeper](https://zookeeper.apache.org/)
* [consul](https://www.consul.io/)
* [confd](https://github.com/kelseyhightower/confd)
* [cfg4j](http://www.cfg4j.org/)
* [super-diamond](https://github.com/melin/super-diamond) ,已停止更新





## 内容来源

[配置中心那点事](http://deadline.top/2016/11/23/%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E9%82%A3%E7%82%B9%E4%BA%8B/)

[一篇好TM长的关于配置中心的文章](http://jm.taobao.org/2016/09/28/an-article-about-config-center/)  
[服务化体系之－配置中心，在ZK或etcd之外](http://calvin1978.blogcn.com/articles/serviceconfig.html)  
[开源分布式配置中心选型](http://vernonzheng.com/2015/02/09/%E5%BC%80%E6%BA%90%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E9%80%89%E5%9E%8B/)  


