# Tomcat的日志

在Tomcat的logs的目录下，通常有以下类似的日志文件

```text
localhost_access_log.2017-12-02.txtlocalhost.2017-12-02.logcatalina.2017-12-02.loghost-manager.2017-12-02.logmanager.2017-12-02.log
```

#### 1、Catalina.log

Catalina引擎（`Catalina is the name of the Tomcat servlet container`）的日志文件，即Tomcat容器自身的日志，文件名catalina.日期.log 

#### 2、localhost.log

Tomcat下内部代码丢出的日志。

#### 3、manager.log

Tomcat自带的manager应用的日志。

#### 4、catalina.out 

控制台输出的日志，Linux下默认重定向到catalina.out。Apache Tomcat中的默认日志记录配置会将日志写入控制台和日志文件catalina.out中。在使用Tomcat进行开发时这非常棒，但通常在生产中不需要。所以，在生产环境中，应该将$CATALINA\_BASE/conf/logging.properties中的`ConsoleHandler`去掉。

```java
handlers = 1catalina.org.apache.juli.AsyncFileHandler, java.util.logging.ConsoleHandler
//改为
handlers = 1catalina.org.apache.juli.AsyncFileHandler
```

#### 5、Access日志

访问日志，即服务器处理的所有的请求的记录，配置在server.xml文件中：

```markup
<!-- Access log processes all example.
      Documentation at: /docs/config/valve.html
      Note: The pattern used is equivalent to using pattern="common" -->
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

  


## 参考

[Logging in Tomcat  
](https://tomcat.apache.org/tomcat-8.5-doc/logging.html#Access_logging)

