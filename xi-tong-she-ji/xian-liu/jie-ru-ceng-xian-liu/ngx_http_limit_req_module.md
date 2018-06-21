# ngx\_http\_limit\_req\_module

### 模块介绍

limit\_req模块是令牌通算法的实现，用于对指定key（一般就是IP）对应的请求数进行限制。它的配置与之前的limit\_conn类似，如下：

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

**配置解释**

limit\_req：配置限流区域、桶（burst）大小，是否延迟（默认延迟）；

limit\_req\_zone：配置限流key、存放key对应信息的共享内存区域大小、固定请求速率；

对应到上面的示例配置，即对客户端IP进行限制，桶大小为5，不延迟，每1s的时间内允许1个请求。

**执行流程**

假如请求在设定的访问频率之内进行访问，那么一切正常。如果请求过快，那么将会对超出设定频率允许的请求做如下处理：

* 默认情况下（没有配置 `burst`, 没有配置 `nodelay`），请求将被直接返回定义的错误码（默认是503）；
* 假如配置了 `burst`，那么超出频率的这些请求将会存放到burst中，但最多只能存放设定的桶大小的数量的请求（放不下的那些请求，将返回错误码）。然后，这些保存的请求将在之后按照设定的访问频率来得到处理。
* 假如配置了 `burst` ，同时也配置了`nodelay`，那么存放到桶中的请求不必等待，将会直接得到处理（此时，处理速率将超过设定的访问频率，即允许突发，但最多突发桶大小的请求数）。

官方文档对`burst` 参数的解释如下：

> Excessive requests are delayed until their number exceeds the maximum burst size \[...\]

官方文档对`nodelay` 参数的解释如下：

> If delaying of excessive requests while requests are being limited is not desired, the parameter nodelay should be used

### 场景理解

#### 场景1

桶容量（burst）为0，延迟模式， 限制每秒500个请求，即固定平均速率为2ms一个请求，配置如下：

```text
limit_req_zone $binary_remote_addr zone=test:10m rate=500r/s
location /limit {
    limit_req zone=test;
    echo "123";
}
```

使用AB测试工具进行测试：并发数2个，总请求100个：

```text
ab -n 100 -c 2 http://localhost/limit
```

> 注意：
>
> ①nginx需要安装echo-nginx-module这个模块才能使用echo指令
>
> ②如果是mac，请将ab测试中的localhost写为127.0.0.1

输入日志部分如下：

\[1529508020.854\]**200** \[1529508020.854\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.855\]503 \[1529508020.856\]**200** \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.856\]503 \[1529508020.857\]503 \[1529508020.857\]503 \[1529508020.857\]503 \[1529508020.857\]503 \[1529508020.857\]503 \[1529508020.857\]503 \[1529508020.858\]**200** \[1529508020.858\]503 \[1529508020.858\]503 \[1529508020.858\]503 \[1529508020.858\]503 \[1529508020.858\]503 \[1529508020.858\]503 \[1529508020.858\]503

通过观察日志，平均每2毫秒允许一次请求；因为桶大小为0，所以：到来的请求，要么直接被限流，要么被正常处理。

假如我们将桶的大小改为2，则

todo：待续





#### 场景2



### 参考

[Nginx模块开发时unknown directive "echo"的处理](https://www.cnblogs.com/chenzhao/p/3939613.html)

[如何安装nginx第三方模块](http://www.ttlsa.com/nginx/how-to-install-nginx-third-modules/)

[nginx添加第三方模块，以及启用nginx本身支持的模块](https://blog.csdn.net/cxm19881208/article/details/64441890)

[Nginx下limit\_req模块burst参数超详细解析](https://blog.csdn.net/hellow__world/article/details/78658041)

[Nginx - what is the meaning to define \`burst\` if there is the \`nodelay\` option](https://serverfault.com/questions/630157/nginx-what-is-the-meaning-to-define-burst-if-there-is-the-nodelay-option)

附测试中最简配置



```text
worker_processes 1;
error_log logs/error.log;
events { 
    worker_connections 1024; 
}
http { 
    log_format main '[$msec]' 
                    '$status'; 
    limit_req_zone $binary_remote_addr zone=test:10m rate=500r/s;
    server {
        listen       8080;
        server_name  localhost;
        access_log  logs/access.log  main;
        location /limit {
            limit_req zone=test;    
            echo "123";
        }
    }
}
```

```text

```

