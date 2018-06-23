# 代理层超时与重试

## Nginx的超时设置

Nginx主要有四类超时设置：客户端超时设置、DNS解析超时设置、代理超时设置，如果使用ngx\_lua，则还有lua相关的超时设置。

### 客户端超时

对于客户端超时主要设置有读取请求头超时时间、读取请求体超时时间、发送响应超时时间、长连接超时时间。通过客户端超时设置避免客户端恶意或者网络状况不佳造成连接长期占用，影响服务端的可处理的能力。

* client\_header\_timeout time：设置读取客户端请求头超时时间，默认为60s，如果在此超时时间内客户端没有发送完请求头，则响应408\(RequestTime-out\)状态码给客户端。
* client\_body\_timeout time：设置读取客户端内容体超时时间，默认为60s，此超时时间指的是两次成功读操作间隔时间，而不是发送整个请求体的超时时间，如果在此超时时间内客户端没有发送任何请求体，则响应408\(RequestTime-out\)状态码给客户端。
* send\_timeout time：设置发送响应到客户端的超时时间，默认为60s，此超时时间指的也是两次成功写操作间隔时间，而不是发送整个响应的超时时间。如果在此超时时间内客户端没有接收任何响应，则Nginx关闭此连接。
* keepalive\_timeout timeout \[header\_timeout\]：设置HTTP长连接超时时间，其中，第一个参数timeout是告诉Nginx长连接超时时间是多少，默认为75s。第二个参数header\_timeout是用于设置响应头“Keep-Alive: timeout=time”，即告知客户端长连接超时时间。两个参数可以不一样，“Keep-Alive:timeout=time”响应头可以在Mozilla和Konqueror系列浏览器起作用，而MSIE长连接默认大约为60s，而不会使用“Keep-Alive: timeout=time”。如Httpclient框架会使用“Keep-Alive: timeout=time”响应头的超时\(如果不设置默认，则认为是永久\)。如果timeout设置为0，则表示禁用长连接。

此参数要配合keepalive\_disable 和keepalive\_requests一起使用。keepalive\_disable 表示禁用哪些浏览器的长连接，默认值为msie6，即禁用一些老版本的MSIE的长连接支持。keepalive\_requests参数作用是一个客户端可以通过此长连接的请求次数，默认为100。



