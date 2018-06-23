# Web容器超时

## Tomcat超时设置

* `connectionTimeout`：配置与客户端建立连接超时时间，从接收到连接后在配置的时间内还没有接收到客户端请求行时，将被认定为连接超时，默认为60000\(60s\)。
* `socket.soTimeout`：从客户端读取请求数据的超时时间，默认同connectionTimeout，NIO和NIO2 支持该配置。
* `asyncTimeout`：Servlet 3异步请求的超时时间，默认为30000\(30s\)。
* `disableUploadTimeout` 和`connectionUploadTimeout`：当配置disableUploadTimeout为false时\(默认为true，和connectionTimeout一样\)，文件上传将使用connectionUploadTimeout作为超时时间。
* `keepAliveTimeout`和`maxKeepAliveRequests`：和`Nginx`配置类似。keepAliveTimeout默认为connectionTimeout，配置-1表示永不超时。maxKeepAliveRequests默认为100。



