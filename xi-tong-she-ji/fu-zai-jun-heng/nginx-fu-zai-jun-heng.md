# Nginx负载均衡（七层）

> 这里Nginx是作为应用负载均衡器，即Nginx直连后端真实应用服务器。

当使用Nginx进行七层负载均衡时，我们一般关心以下几个方面：

* **上游服务器配置**：使用upstream server配置上游服务器。
* **负载均衡算法**：配置多个上游服务器时的负载均衡机制。
* **失败重试机制**：配置当超时或上游服务器不存活时，是否需要重试其他上游服务器。
* **服务器心跳检查**：上游服务器的健康检查/心跳检查。

## Nginx上游服务器配置

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

①`down`：表示当前server暂时不参与负载均衡。当测试或者某个后端服务器出现故障时，可以暂时通过该配置临时摘掉机器。  
②`backup`：表示是预留的备份机，当其他所有非backup机器出现故障或者繁忙的时候，才会请求backup机器，这台机器的访问压力最轻。如通过缩容上游服务器进行压测时，要摘掉一些上游服务器进行压测，但为了保险起见会配置一些备上游服务器，当压测的上游服务器都挂掉时，流量可以转发到备上游服务器，从而不影响用户请求处理。  


```lua
upstream backend {
        server 192.168.0.100:8080 ;
        server 192.168.0.101:8080 ;
        server 192.168.0.101:8080 down;
        server 192.168.0.101:8080 backup;
}
```

> 还有max\_fails和fail\_timeout等参数，后面的失败重试部分再进行说明。

## Nginx负载均衡算法

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

## Nginx失败重试

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

详见：[代理层超时与重试](https://maxwell.gitbook.io/way-to-architect/~/edit/drafts/-LFmi05RkCD7vKp_K5tp/xi-tong-she-ji/chao-shi-yu-zhong-shi-ji-zhi/dai-li-ceng-chao-shi-yu-zhong-shi)

## Nginx健康检查

Nginx对上游服务器的健康检查默认采用的是惰性策略（Nginx商业版提供了进行主动健康检查）。当然也可以集成[nginx\_upstream\_check\_module](https://github.com/yaoweibin/nginx_upstream_check_module)模块来进行主动健康检查。

[nginx\_upstream\_check\_module](https://github.com/yaoweibin/nginx_upstream_check_module)模块支持以下两种检查模式。

### 1、**TCP心跳检查**

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

### 2、**HTTP心跳检查**

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

## Nginx与上游服务器的长连接

通过如下配置：

```lua
http {
    ... ...
    upstream backend {
    　　server 192.168.61.1:9080 weight=1;
    　　server 192.168.61.1:9090 weight=2 backup;
    　　keepalive 100;
    }
    server {
        listen 80;
        location / {
        　  #http1.1支持keep-alive
        　　proxy_http_version 1.1;
        　　proxy_set_header Connection "";
        　　proxy_pass http://backend;
　　    }
    }
    ... ...
}
```

配置说明：

`keepalive`：配置长连接数量，这里是指每个Worker进程与上游服务器可缓存的空闲连接的最大数量（不是总连接数，keepalive指令不限制Worker进程与上游服务器的总连接数量）。当超出这个数量时，最近最少使用的连接将被关闭。

> ①如果是http/1.0，则需要配置发送“Connection: Keep-Alive”请求头。
>
> ②上游服务器不要忘记开启长连接支持。

## Nginx反向代理配置

Nginx在七层的作用就是反向代理，它除了实现了负载均衡之外，还提供如缓存来减少上游服务器的压力。

### 全局配置缓存

```lua
http {
   ... ...
   proxy_buffering on;
　　proxy_buffer_size 4k;
　　proxy_buffers 512 4k;
　　proxy_busy_buffers_size 64k;
　　proxy_temp_file_write_size 256k;
　　proxy_cache_lock on;
　　proxy_cache_lock_timeout 200ms;
　　proxy_temp_path /tmpfs/proxy_temp;
　　proxy_cache_path /tmpfs/proxy_cache levels=1:2keys_zone =cache:512m inactive=5m max_size=8g;
　　proxy_connect_timeout 3s;
　　proxy_read_timeout 5s;
　　proxy_send_timeout 5s;

   gzip on;
　　gzip_min_length 1k;
　　gzip_buffers 16 16k;
　　gzip_http_version 1.0;
　　gzip_proxied any;
　　gzip_comp_level 2;
　　gzip_types text/plainapplication/x-java text/css application/xml;
　　gzip_vary on;

   server{
      ... ...
   }
   ... ...
}
    
```

配置说明：

①开启proxy buffer，缓存内容将存放在tmpfs（内存文件系统）以提升性能，设置超时时间。

②开启gzip支持，减少网络传输的数据包大小。对于内容型响应建议开启gzip压缩，gzip\_comp\_level压缩级别要根据实际压测来决定（带宽和吞吐量之间的抉择）。

### location配置缓存

```lua
location ~ ^/backend/(.*)$ {
　　#请求上游服务器使用GET方法（不管请求是什么方法）
　　proxy_method GET;
　　#不给上游服务器传递请求体
　　proxy_pass_request_body off;
　　#不给上游服务器传递请求头
　　proxy_pass_request_headers off;
　　#设置上游服务器的哪些响应头不发送给客户端
　　proxy_hide_header Vary;
　　#支持keep-alive
　　proxy_http_version 1.1;
　　proxy_set_header Connection "";
　　#给上游服务器传递Referer、Cookie和Host（按需传递）
　　proxy_set_header Referer $http_referer;
　　proxy_set_header Cookie $http_cookie;
　　proxy_set_header Host web.c.3.local;
　　proxy_pass http://backend /$1$is_args$args;
}
```

上述配置中开启了`proxy_pass_request_body`和`proxy_pass_request_headers`，禁止向上游服务器传递请求头和内容体，从而使得上游服务器不受请求头攻击，也不需要解析；如果需要传递，则使用proxy\_set\_header按需传递即可。

## 参考

《亿级流量网站架构核心技术》：负载均衡与反向代理

[Nginx几种负载均衡算法及配置实例](https://www.jianshu.com/p/129fe671deed)

[Nginx缓存设置](http://blog.51cto.com/linux008/547236)

