# ngx\_http\_limit\_req\_module

limit\_req模块是漏桶算法的实现，用于对指定key（一般就是IP）对应的请求数进行限制。它的配置与之前的limit\_conn类似，如下：

```text
http{
    limit_req_zone #binary_remote_addr zone=perip:10m rate=1r/s;
    limit_conn_log_level error;
    limit_conn_status 503;
    
    server{
        location / {
            limit_req zone=perip burst=5 nodelay;
        }
    }
}
```

配置解释：

limit\_req：配置限流区域、桶（burst）大小，是否延迟（默认延迟）；

limit\_req\_zone：配置限流key、存放key对应信息的共享内存区域大小、固定请求速率；

对应到上面的示例配置，即对客户端IP进行限制，桶大小为5，不延迟，每1s的时间内允许1个请求。

执行流程：

①请求进入后首先判断最后一次请求时间相对于当前时间是否需要限流，需要限流则执行②，不需要限流则执行③；

②如果没有配置桶容量，默认为0，按照固定速率处理请求，如果请求被限流，则直接返回错误码；

如果配置了桶大小且为延迟模式（没有写明nodelay），如果桶满了，则新进入的请求被限流。如果没有满，则请求会以固定平均速率被处理。

如果配置了桶大小且为非延迟模式（写明nodelay），则不会按照固定速率处理请求，而是允许突发流量请求。如果桶满了，则请求被限流，直接返回相应的错误码。

③如果没有限流，则正常处理请求；

④Nginx会在相应时机选择一些限流key进行过期处理，进行内存回收。



