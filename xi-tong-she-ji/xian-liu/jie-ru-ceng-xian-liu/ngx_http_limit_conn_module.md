# ngx\_http\_limit\_conn\_module

limit\_conn是对某个key对应的总的网络连接数进行限制：可以按照IP来限制IP维度的总连接数，也可以按照服务域名来限制某个域名的总连接数。

但是，不是每个请求连接都会被计数器统计，只有那些被Nginx处理的且已经读取了整个请求头的请求连接才会被计数器统计。

### 配置示例

```text
http{
    # 使用客户端的IP（$binary_remote_addr）作为key，
    # zone=perip:10m的意思是定义名为perip（名字任意）的共享内存区间，区间大小为10M
    # 由于$binary_remote_addr固定为4个字节，存储状态在32位平台中占用32字节或64字节，在64位平台中占用64字节，
    # 所以，一共可以存储num=[10*1000/（4+32）或 10*1000/（4+64）]个状态
    # 该配置可以记录num个<客户端IP,连接数>这样的键值对
    # 也就是说，一共允许num个客户端IP进行连接
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    # 当达到最大限制连接数后，记录日志的等级。
    limit_conn_log_level error;
    # 如果共享内存空间被耗尽，服务器将会对后续所有的请求返回503 
    limit_conn_status 503;
    ...
    server{
        ...
        location /{
            # 指定每个给定键值（这里就是指固定的一个客户端IP）的最大同时连接数为1
            # 即同一客户端IP同一时间只允许有1个连接
            # 如果这里配置为2，则表示同一客户端IP同一时间允许有2个连接，
            # 也就是允许该IP的并发连接数为2
            limit_conn perip 1;
        }
    }
}
```

执行流程：①请求进入后，首先判断当前limit\_conn\_zone中相应的key（这里就是客户端IP）的连接数是否超出了配置的最大连接数（即1）；②如果超出了配置的最大连接数，则被限流，返回limit\_conn\_status定义的错误码（这里就是503），否则，相应的key的连接数加1，并注册请求处理完成的回调函数；③进行请求处理；④在结束请求时，会调用注册的回调函数，对相应的key的连接数减1。

注意：如果按照上面的配置，则允许的总的最大网络连接数为：num \* 1

**限制虚拟主机的总连接数的配置**

如果需要按照域名（或虚拟主机）进行限制，则需要先定义一个域名维度的共享内存区域：

```text
limit_conn_zone $server_name zone=perserver:10m;
```

然后，假如限制每个域名只能同时允许1000个连接数，则在相应的location内添加：

```text
limit_conn perserver 1000;
```

完整配置如下

```text
http{

    limit_conn_zone $server_name zone=perserver:10m;
    
    server {
        listen 8000;
        server_name www.xxxxx.com xxxxx.com;
        location / {
            ...
            limit_conn perserver 1000;
        }
    }
    server {
        listen 8600;
        server_name www.yyyyyy.com yyyyy.com;
        location / {
            ...
            limit_conn perserver 1000;
        }
    }
}
```

**同时限制ip和虚拟主机最大连接数的配置**

```text
http {
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    limit_conn_zone $server_name zone=perserver:10m;
    server {
        location / {
            limit_conn perip 10;
            limit_conn perserver 1000;
        }
    }
}
```

此时，所有的连接数限制都会生效。上面的配置不仅会限制单一IP来源的连接数，同时也会限制单一虚拟服务器的总连接数。

### 限制连接的网络传输速率

此外，该模块还有一个配置为：`limit_rate rate`，默认rate为0，可以配置在：http, server, location, if in location这些地方。该配置是对每个连接的速率限制。参数rate的单位是字节/秒，设置为0将关闭限速。 按连接限速而不是按IP限制，因此如果某个客户端同时开启了两个连接，那么客户端的整体速率是这条指令设置值的2倍。

```text
http { 
    limit_conn_zone $binary_remote_addr zone=perip:10m; 
    limit_conn_log_level info;  
    server { 
        location  ^~ /download/ {   
            limit_conn perip 4; 
            limit_rate 200k;                   
        } 
    }
}
```

### 配置白名单

```text
http {  
    geo $whiteiplist {        
        default 1;        
        127.0.0.1 0;        
        10.10.0.0/24 0;    
    }    
    map $whiteiplist $limit {        
        1 $binary_remote_addr;        
        0 "";    
    } 
       
    limit_conn_zone $limit zone=perip:10m;    
    
    server {
        limit_conn perip 5;
    }
}
```

说明：  
1. geo指令定义一个白名单`$whiteiplist`, 默认值为1, 所有都受限制。 如果客户端IP与白名单列出的IP相匹配，则`$whiteiplist`值为0也就是不受限制。  
2. map指令是将`$whiteiplist`值为1的，也就是受限制的IP，映射为客户端IP。将`$whiteiplist`值为0的，也就是白名单IP，映射为空的字符串。  
3. `limit_conn_zone`（以及`limit_req_zone`）指令对于键为空值的将会被忽略，从而实现对于列出来的IP不做限制。

### 注意事项

ngx\_http\_limit\_conn\_module 模块虽然按照IP/虚拟主机等维度进行限制总连接数或网络传输速率，但是会引入另外一些问题的：如前端如果有做LVS或反向代理，而我们后端启用了该模块功能，就会有非常多503错误了。因此，可以在最前端的Nginx中启用该模块，或者设置白名单。

## 参考

《亿级流量网站架构核心技术》：限流详解

[nginx限制连接数ngx\_http\_limit\_conn\_module模块](http://www.ttlsa.com/nginx/nginx-limited-connection-number-ngx_http_limit_conn_module-module/)

[Nginx连接数、请求数限制应用详解](https://segmentfault.com/a/1190000004688125)

[Nginx虚拟主机配置](https://www.jianshu.com/p/4ef8a7a374f3)

[Nginx通过geo模块设置白名单](https://blog.whsir.com/post-2387.html)





