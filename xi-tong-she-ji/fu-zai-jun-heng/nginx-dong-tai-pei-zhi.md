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

### 使用Consul + Consul-template实现

整体架构图如下：

![](../../.gitbook/assets/image%20%284%29.png)

1．首先，`upstream`服务启动，我们通过管理后台向`Consul`注册服务。

2．在`Nginx`机器上部署并启动`Consul-template Agent`，其通过长轮询监听服务变更。

3．`Consul-template`监听到变更后，动态修改`upstream`列表。

4．`Consul-template`修改完`upstream`列表后，调用重启`Nginx`脚本重启`Nginx`。

首先我们要启动Consul-Server。

./consul agent -server -bootstrap-expect 1-data-dir /tmp/consul  -bind 0.0.0.0-client 0.0.0.0

此处需要使用data-dir指定Agent状态存储位置，

bind指定集群通信的地址，

client指定客户端通信的地址（如Consul-template与Consul通信）。

在启动时还可以使用-ui-dir ./ui/指定Consul Web UI目录，实现通过Web UI管理Consul，

然后访问如http://127.0.0.1:8500即可看到控制界面。

![](http://www.10tiao.com/img.do?url=http%3A//mmbiz.qpic.cn/mmbiz_png/G38RbhwficlcMyKCicVMJssAEwcDwJ0VZvPsczTI4IPuEKfBN3WA17XFp0FXoOOpj1GydRicc1lUX4H7A9zQBQxMQ/0%3Fwx_fmt%3Dpng)

使用如下HTTP API注册服务。

```bash
curl -X PUT http://127.0.0.1:8500/v1/catalog/register -d'{"Datacenter": "dc1", "Node":"tomcat", "Address":"192.168.1.1","Service": {"Id" :"192.168.1.1:8080", "Service": "item_jd_tomcat","tags": ["dev"], "Port": 8080}}'
curl -X PUT http://127.0.0.1:8500/v1/catalog/register -d'{"Datacenter": "dc1", "Node":"tomcat", "Address":"192.168.1.2","Service": {"Id" :"192.168.1.1:8090", "Service": "item_jd_tomcat","tags": ["dev"], "Port": 8090}}'
```

上面指令中，Datacenter指定数据中心，Address指定服务IP，Service.Id指定服务唯一标识，Service.Service指定服务分组，Service.tags指定服务标签（如测试环境、预发环境等），Service.Port指定服务端口。

通过如下HTTP API摘除服务。

```bash
curl -X PUThttp://127.0.0.1:8500/v1/catalog/deregister -d '{"Datacenter":"dc1", "Node": "tomcat", "ServiceID" :"192.168.1.1:8080"}'
```

通过如下HTTP API发现服务。

```bash
curl http://127.0.0.1:8500/v1/catalog/service/item_jd_tomcat
```

接下来我们需要在Consul-template机器上添加一份配置模板`item.jd.tomcat.ctmpl`。

```lua
upstream item_jd_tomcat {
    #占位server，必须有一个server，否则无法启动
    server 127.0.0.1:1111; 
    {{range service "dev.item_jd_tomcat@dc1"}}
       server {{.Address}}:{{.Port}} weight=1;
    {{end}}
}
```

service 指定格式为：标签.服务@数据中心，然后通过循环输出Address和Port，从而生成Nginx upstream配置。

接下来启动Consul-template：

```bash
./consul-template -consul 127.0.0.1:8500 -template./item.jd.tomcat.ctmpl:/usr/servers/nginx/conf/domains/item.jd.tomcat:"./restart.sh"
```

使用consul指定Consul服务器客户端通信地址，template格式是“配置模板:目标配置文件:脚本”，即通过配置模板更新目标配置文件，然后调用脚本重启Nginx。

直接通过`Nginx include`指令将`/usr/servers/nginx/conf/domains/item.jd.tomcat`包含到`nginx.conf`配置文件即可，`restart.sh`脚本代码如下所示。

```bash
#!/bin/bash
ps -fe | grep nginx | grep -v grep
if [ $? -ne 0 ]
then
  sudo /usr/servers/nginx/sbin/nginx
  echo "nginx start"
else
  sudo /usr/servers/nginx/sbin/nginx -s reload
  echo "nginx reload"
fi
```

即如果Nginx没有启动，则启动，否则重启。

Java真实服务中进行服务注册与摘除

项目中添加`Consul Java Client`：

```markup
<dependency>
    <groupId>com.orbitz.consul</groupId>
    <artifactId>consul-client</artifactId>
    <version>0.12.8</version>
</dependency>
```

代码中进行服务注册与摘除：

```java
public static void main(String[] args) {
    //启动嵌入容器（如Tomcat）
    SpringApplication.run(Bootstrap.class, args);
    //服务注册
    Consul consul = Consul.builder().withHostAndPort(HostAndPort.fromString ("192.168.61.129:8500")).build();
    final AgentClient agentClient = consul.agentClient();
    String service = "item_jd_tomcat";
    String address = "192.168.61.1";
    String tag = "dev";
    int port= 9080;
    final String serviceId = address + ":" + port;
    ImmutableRegistration.Builder builder = ImmutableRegistration.builder();
    builder.id(serviceId).name(service).address(address).port(port).addTags(tag);
    agentClient.register(builder.build());
    //JVM停止时摘除服务
    Runtime.getRuntime().addShutdownHook(new Thread() {
       @Override
        publicvoid run() {
           agentClient.deregister(serviceId);
        }
    });
}
```

到此我们就实现了动态`upstream`负载均衡，`upstream`服务启动后自动注册到`Nginx`，`upstream`服务停止时，自动从`Nginx`上摘除。

通过`Consul`+ `Consul-template`方式，每次发现配置变更都需要`reload nginx`，而reload是有一定损耗的。而且，如果你需要长连接支持的话，那么当`reload nginx`时长连接所在worker进程会进行优雅退出，并当该worker进程上的所有连接都释放时，进程才真正退出（表现为worker进程处于worker process is shutting down）。因此，如果能做到不reload就能动态更改upstream，那么就完美了。对于社区版Nginx目前有三个选择：Tengine的Dyups模块、微博的Upsync和使用OpenResty的balancer\_by\_lua。微博使用Upsync+Consul实现动态负载均衡，而又拍云使用其开源的slardar（Consul + balancer\_by\_lua）实现动态负载均衡。

### 使用**Consul+OpenResty实现**

todo

## 内容来源

[使用Nginx实现HTTP动态负载均衡—《亿级流量网站架构核心技术》  
](http://www.10tiao.com/html/164/201703/2652898370/1.html)





###  

