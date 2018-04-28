# 基本配置

`nginx.conf`是`Nginx`的配置文件，其文件结构如下

```text
### 全局模块                                      
...

### events模块
events {                                                 
    ...
}

### http模块
http {
    ...

    server {
        ...
        location /web1 {
          ...
        }
        location /web2 {
          ...
        }
    }
    server {
        ...
        ...
    }
}
```

  


