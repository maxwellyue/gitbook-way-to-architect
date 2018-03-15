Tomcat生产环境中的配置

Tomcat的配置或者优化，主要从两方面考虑：JVM和Tomcat自身。

### JVM

---

Tomcat首先跑在JVM之上的，首先我们需要对这个JVM实例进行调优。

这些修改在`catalina.bat`或者`catalina.sh`文件中。

①**-server：让tomcat以server模式运行。**

因为tomcat默认是以一种叫java –client的模式来运行的，server即意味着你的tomcat是以真实的production的模式在运行的，这也就意味着你的tomcat以server模式运行时将拥有：更大、更高的并发处理能力，更快更强捷的JVM垃圾回收机制，可以获得更多的负载与吞吐量。

②**-Xms–Xmx：设置堆的初始大小和最大值。**

③**–Xmn：**Sun官方推荐配置为整个堆的**3/8**。

④ **-Xss：**是指设定每个线程的栈大小。这个就要依据你的程序，看一个线程 大约需要占用多少内存，可能会有多少线程同时运行等。一般不易设置超过1M，要不然容易出现out ofmemory。

其他地可以参考JVM优化的一般做法。

### Tomcat容器

---

对 tomcat 来说，每一个进来的请求（request）都需要一个线程，直到该请求结束。如果同时进来的请求多于当前可用的请求处理线程数，额外的线程就会被创建，直到到达配置的最大线程数（maxThreads 属性值）。如果仍就接收到更多请求，这些来不及处理的请求就会在 Connector 创建的 Server Socket 中堆积起来，直到到达最大的配置值（acceptCount 属性值）。至此，任何再来的请求将会收到 connection refused 错误，直到有可用的资源来处理它们。

  


这些修改是在`/conf/server.xml`文件中。

首先，看文件中Connector的默认设置：

`<Connector port="8080" protocol="HTTP/1.1"/>`

修改为如下：

```
<Connector port="8080" protocol="HTTP/1.1"

          URIEncoding="UTF-8"    使tomcat解析含有中文名的文件的url

          minSpareThreads="25"   如果空闲状态的线程数多于该值，则将这些线程中止，减少这个池中的线程总数。

          maxSpareThreads="75"   最小备用线程数，tomcat启动时的初始化的线程数

          enableLookups="false"  如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址

          connectionTimeout="30000"  网络连接超时时间毫秒数，设置为0表示永不超时，但这样设置有隐患的。

          acceptCount="300"  当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，如果这个队列也满了，就直接refuse connection

          maxConnections:    tomcat同时可建立的最大连接数
          
          maxThreads="300"  实际可同时处理的请求数

          compression="on" 配置gzip压缩(HTTP压缩)功能

          compressionMinSize="2048"

          compressableMimeType="text/html,text/xml,text/JavaScript,text/css,text/plain"

          redirectPort="8443"
/>
```

---

内容源自：

[nginx 和 tomcat 生产环境配置 建议和方法](http://ihuangweiwei.iteye.com/blog/1233941)

[tomcat 7 连接数和线程数配置](https://www.jianshu.com/p/8445645b3aff)

