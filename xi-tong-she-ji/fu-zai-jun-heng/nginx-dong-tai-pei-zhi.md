# Nginx动态配置

## Nginx七层负载均衡动态配置

在使用upstream实现负载均衡时，假如upstream列表有变更，那么就需要到服务器进行手动修改。有没有这样一种方式：后端服务上线时，将其自动注册到upstream列表中；后端服务下线时，自动从upstream列表中摘除？答案就是使用配置中心。

这里以Consul为例。

Consul是一款开源的分布式服务注册与发现系统，通过HTTP API可以使得服务注册、发现实现起来非常简单，它支持如下特性：

* **服务注册**：服务实现者可以通过HTTP API或DNS方式，将服务注册到Consul。
* **服务发现**：服务消费者可以通过HTTP API或DNS方式，从Consul获取服务的IP和PORT。
* **故障检测**：支持如TCP、HTTP等方式的健康检查机制，从而当服务有故障时自动摘除。
* **K/V存储**：使用K/V存储实现动态配置中心，其使用HTTP长轮询实现变更触发和配置更改。
* **多数据中心**：支持多数据中心，可以按照数据中心注册和发现服务，即支持只消费本地机房服务，使用多数据中心集群还可以避免单数据中心的单点故障。
* **Raft算法**：Consul使用Raft算法实现集群数据一致性。

通过Consul可以管理服务注册与发现，接下来需要有一个与Nginx部署在同一台机器的Agent来实现Nginx配置更改和Nginx重启功能。我们有Confd或者Consul-template两个选择，而Consul-template是Consul官方提供的，我们就选择它了。其使用HTTP长轮询实现变更触发和配置更改（使用Consul的watch命令实现）。也就是说，我们使用Consul-template实现配置模板，然后拉取Consul配置渲染模板来生成Nginx实际配置。

除Consul外，还有一个选择是etcd3，其使用了gRPC和protobuf可以说是一个亮点。不过，etcd3目前没有提供多数据中心、故障检测、Web管理界面。

### Consul + Consul-template

整体架构图如下：

![](../../.gitbook/assets/image%20%284%29.png)

1．首先，upstream服务启动，我们通过管理后台向Consule注册服务。

2．在Nginx机器上部署并启动Consul-template Agent，其通过长轮询监听服务变更。

3．Consul-template监听到变更后，动态修改upstream列表。

4．Consul-template修改完upstream列表后，调用重启Nginx脚本重启Nginx。

  






###  

