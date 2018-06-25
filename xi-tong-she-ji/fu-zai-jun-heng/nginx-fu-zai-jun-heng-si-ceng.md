# Nginx负载均衡（四层）

Nginx 从`1.9.0`版本起支持四层负载均衡，从而使得Nginx变得更加强大。目前，四层软件负载均衡器用得比较多的是HaProxy；而Nginx也支持四层负载均衡，一般场景我们使用Nginx一站式解决方案就够了。

## **静态负载均衡**

在默认情况下，`ngx_stream_core_module` 是没有启用的，需要在安装Nginx时，添加--with-stream配置参数启用。

```text
./configure --prefix=/usr/servers --with-stream
```

### 1.**stream指令**

我们配置HTTP负载均衡时，都是配置在http指令下，而四层负载均衡则是配置在stream指令下。

```lua
stream {
    upstream backend {
        ... ...
    }
    server {
        ... ...
    }
}
```

### 2.**upstream配置**

类似于http upstream配置，配置如下：

```lua
upstream backend {
    server 192.168.0.10:3306 max_fails=2 fail_timeout=10s weight=1;
    server 192.168.0.11:3306 max_fails=2 fail_timeout=10s weight=1;
    least_conn;
}
```

失败重试、惰性健康检查、负载均衡算法的相关配置与HTTP负载均衡配置类似。此处我们配置实现了两个数据库服务器的TCP负载均衡。

### 3.**server配置**

```lua
server {
    #监听端口
    listen 3308;
    #失败重试
    proxy_next_upstream on;
    proxy_next_upstream_timeout 0;
    proxy_next_upstream_tries 0;
    #超时配置
    proxy_connect_timeout 1s;
    proxy_timeout 1m;
    #限速配置
    proxy_upload_rate 0;
    proxy_download_rate 0;
    #上游服务器
    proxy_pass backend;
}
```

配置说明：

①`listen`指令指定监听的端口，默认`TCP`协议，如果需要`UDP` ，则可以配置`listen 3308 udp;`。

②`proxy_next_upstream`\*与HTTP负载均衡类似。

③`proxy_connect_timeout`配置与上游服务器连接超时时间，默认60s。

④`proxy_timeout`配置与客户端或上游服务器连接的两次成功读/写操作的超时时间，如果超时，将自动断开连接，即连接存活时间，通过它可以释放那些不活跃的连接，默认10分钟。

⑤`proxy_upload_rate`和`proxy_download_rate`分别配置从客户端读数据和从上游服务器读数据的速率，单位为每秒字节数，默认为0，不限速。

接下来就可以连接Nginx的3308端口，访问我们的数据库服务器了。

目前的配置都是静态配置，像数据库连接一般都是使用长连接，如果重启Nginx服务器，则会看到如下Worker进程一直不退出。

```text
Nobody 10268 ……nginx: worker process is shutting down
```

这是因为Worker维持的长连接一直在使用，所以无法退出，解决办法只能是杀掉该进程。

当然，一般情况下是因为需要动态添加/删除上游服务器，才需要重启Nginx，像HTTP动态负载均衡那样。如果能做到动态负载均衡，则一大部分问题就解决了。

## **动态负载均衡**

todo



内容来源

《亿级流量网站架构核心技术》：负载均衡与反向代理

