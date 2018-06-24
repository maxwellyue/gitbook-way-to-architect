# Nginx负载均衡

> 这里Nginx是作为应用负载均衡器，即Nginx直连后端真实应用服务器。

当使用Nginx进行七层负载均衡时，我们一般关心以下几个方面：

* **上游服务器配置**：使用upstream server配置上游服务器。
* **负载均衡算法**：配置多个上游服务器时的负载均衡机制。
* **失败重试机制**：配置当超时或上游服务器不存活时，是否需要重试其他上游服务器。
* **服务器心跳检查**：上游服务器的健康检查/心跳检查。

### Nginx上游服务器配置

通过在http指令下配置upstream即可。

```lua
http {
    ... ...
    upstream backend {
        server 192.168.0.100:8080 ;
        server 192.168.0.101:8080 ;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
    ... ...
}
```

如上所示，可以通过server命令通过指定IP地址和端口来访问后端服务器。同时，也支持域名的方式，Nginx会进行域名解析（TODO）。

同时，可以设置后端服务器的状态：

①down：表示当前server暂时不参与负载均衡。

②backup：表示是预留的备份机，当其他所有非backup机器出现故障或者繁忙的时候，才会请求backup机器，这台机器的访问压力最轻。

```lua
upstream backend {
        server 192.168.0.100:8080 ;
        server 192.168.0.101:8080 ;
        server 192.168.0.101:8080 down;
        server 192.168.0.101:8080 backup;
}
```

> 还有max\_fails和fail\_timeout等参数，后面的失败重试部分再进行说明。

### Nginx负载均衡算法

Nginx中支持的负载均衡算法（用户请求到来时选择哪个后端服务器）主要有以下几种：

#### **轮询（**`round-robin`**），默认的负载均衡算法**

以轮询的方式将请求转发到上游服务器，通过配合`weight` 配置可以实现基于权重的轮询。

#### 最少连接（`least_conn`）

将请求负载均衡到最少活跃连接的上游服务器。如果配置的服务器较少，则将转而使用基于权重的轮询算法。

#### 最小响应时间（`least_time`）

Nginx商业版支持该算法，基于最小平均响应时间进行负载均衡。

#### 公平（`fair`，第三方）

根据后端服务器的响应时间来分配请求，响应时间短的优先分配。如果想要使用此调度算法，需要Nginx的`upstream_fair`模块。

#### IP哈希（`ip_hash`）

根据客户IP进行负载均衡，即相同的IP将负载均衡到同一个Upstream Server。　　

#### URL哈希（`url_hash`，第三方）

按访问URL的哈希结果来分配请求，使每个URL定向到同一台后端服务器，可以进一步提高后端缓存服务器的效率。如果想要使用此调度算法，需要Nginx的hash软件包。

#### 哈希（`hash key [consistent]`）

对某一个key进行哈希或者使用一致性哈希算法进行负载均衡。使用Hash算法存在的问题是，当添加/删除一台服务器时，将导致很多key被重新负载均衡到不同的服务器（从而导致后端可能出现问题）；因此，建议考虑使用一致性哈希算法，这样当添加/删除一台服务器时，只有少数key将被重新负载均衡到不同的服务器。

TODO：各种算法的配置示例

### Nginx失败重试

主要有两部分配置：upstream server和proxy\_pass。

```lua
http {
    ... ...
    upstream backend {
    　　server 192.168.61.1:9080 max_fails=2 fail_timeout=10s weight=1;
    　　server 192.168.61.1:9090 max_fails=2 fail_timeout=10s weight=1;
    }
    
    server {
        listen 80;
        location /test {
        　　proxy_connect_timeout 5s;
        　　proxy_read_timeout 5s;
        　　proxy_send_timeout 5s;
        　　proxy_next_upstreamerror timeout;
        　　proxy_next_upstream_timeout 10s;
        　　proxy_next_upstream_tries 2;
        　　proxy_pass http://backend;
        　　add_header upstream_addr $upstream_addr;
　　    }
　　}
    ... ...
}
```

上游服务器中的`max_fails`和`fail_timeout`：指定每个上游服务器，当`fail_timeout`时间内失败了`max_fails`次请求，则认为该上游服务器不可用/不存活，然后将摘掉该上游服务器，`fail_timeout`时间后会再次将该服务器加入到存活上游服务器列表进行重试。

todo

### Nginx健康检查

Nginx对上游服务器的健康检查默认采用的是惰性策略（Nginx商业版提供了进行主动健康检查）。当然也可以集成[nginx\_upstream\_check\_module](https://github.com/yaoweibin/nginx_upstream_check_module)模块来进行主动健康检查。

[nginx\_upstream\_check\_module](https://github.com/yaoweibin/nginx_upstream_check_module)模块支持以下两种检查模式。

**TCP心跳检查**

```lua
upstream backend {
　　server 192.168.61.1:9080 weight=1;
　　server 192.168.61.1:9090 weight=2;
　　check interval=3000 rise=1 fall=3 timeout=2000 type=tcp;
}
```

参数说明：

　　interval：检测间隔时间，此处配置了每隔3s检测一次。

　　fall：检测失败多少次后，上游服务器被标识为不存活。

　　rise：检测成功多少次后，上游服务器被标识为存活，并可以处理请求。

　　timeout：检测请求超时时间配置。

**HTTP心跳检查**

```lua
upstream backend {
　　server 192.168.61.1:9080 weight=1;
　　server 192.168.61.1:9090 weight=2;
　　check interval=3000 rise=1 fall=3 timeout=2000 type=http;
　　check_http_send "HEAD /status HTTP/1.0rnrn";
　　check_http_expect_alive http_2xx http_3xx;
}
```

参数说明：（未说明参数与TCP心跳检查部分一致）

　　check\_http\_send：即检查时发的HTTP请求内容。

　　check\_http\_expect\_alive：当上游服务器返回匹配的响应状态码时，则认为上游服务器存活。

此处需要注意，检查间隔时间不能太短，否则可能因为心跳检查包太多造成上游服务器挂掉，同时要设置合理的超时时间。



  




## 参考

《亿级流量网站架构核心技术》：负载均衡与反向代理

[Nginx几种负载均衡算法及配置实例](https://www.jianshu.com/p/129fe671deed)

