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

这些修改是在`/conf/server.xml`文件中。

首先，看文件中Connector的默认设置：

`<Connector port="8080" protocol="HTTP/1.1"/>`

修改为如下：

```
<Connector port="8080" protocol="HTTP/1.1"

          URIEncoding="UTF-8"    使tomcat解析含有中文名的文件的url

          minSpareThreads="25"   如果空闲状态的线程数多于该值，则将这些线程中止，减少这个池中的线程总数。

          maxSpareThreads="75"   最小备用线程数，tomcat启动时的初始化的线程数

          enableLookups="false"  是否反查域名。为了提高处理能力，应设置为 false

          connectionTimeout="30000"  网络连接超时时间毫秒数，设置为0表示永不超时，但这样设置有隐患的。

          acceptCount="300"  当线程数达到maxThreads后，后续请求会被放入一个等待队列，这个acceptCount是这个队列的大小，如果这个队列也满了，就直接refuse connection

          maxThreads="300"   

          maxProcessors="1000" 

          minProcessors="5"

          useURIValidationHack="false" 

          compression="on" 配置gzip压缩(HTTP压缩)功能

          compressionMinSize="2048"

          compressableMimeType="text/html,text/xml,text/JavaScript,text/css,text/plain"

          redirectPort="8443"
/>
```



