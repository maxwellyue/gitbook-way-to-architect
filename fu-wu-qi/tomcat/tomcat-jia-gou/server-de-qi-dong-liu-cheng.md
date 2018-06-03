# Server 的启动流程

#### 服务器启动 {#Server_Startup}

本节介绍Tomcat服务器如何启动。启动tomcat有几种不同的方式，包括：

* 从命令行。
* 从Java程序作为嵌入式服务器。
* 自动作为Windows服务。

**描述**

启动程序的文本说明可[在此处找到。](http://tomcat.apache.org/tomcat-8.5-doc/architecture/startup/serverStartup.txt)

其中的内容如下：

```text
Tomcat启动顺序

1.从命令行启动
类：org.apache.catalina.startup.Bootstrap
它能做什么：
    a）设置类加载器
        commonLoader（通用） - >系统加载程序
        sharedLoader（共享） - > commonLoader  - >系统加载器
        catalinaLoader（服务器） - > commonLoader  - >系统加载程序
        （默认情况下，commonLoader用于sharedLoader和serverLoader）
    b）加载启动类（反射）
        org.apache.catalina.startup.Catalina
        setParentClassloader  - > sharedLoader
        Thread.contextClassloader  - > catalinaLoader
    c）Bootstrap.daemon.init（）完成

顺序2.处理命令行参数（开始，停止）
Class：org.apache.catalina.startup.Bootstrap（假设command-> start）
它能做什么：
    a）Catalina.setAwait（true）;
    b）Catalina.load（）
        b1）initDirs（） - >设置属性，比如catalina.home
                          catalina.base == catalina.home（大多数情况下）
        b2）initNaming
            setProperty（javax.naming.Context.INITIAL_CONTEXT_FACTORY，
                    org.apache.naming.java.javaURLContextFactory  - > default）
        b3）createStartDigester（）
            为主要server.xml元素配置一个消化器
            org.apache.catalina.core.StandardServer（当然可以改变:)
            org.apache.catalina.deploy.NamingResources
                在J2EE JNDI树中存储命名资源
            org.apache.catalina.LifecycleListener
                实现主要组件启动/停止的事件
            org.apache.catalina.core.StandardService
                一组连接器的单个条目，
                这样一个容器就可以收听多个连接器
                即单个条目
            org.apache.catalina.Connector
                连接器仅侦听传入请求
            它还将以下规则集添加到蒸煮器中
                NamingRuleSet
                EngineRuleSet
                HostRuleSet
                ContextRuleSet
        b4）加载server.xml并使用消化器进行解析
            使用消化器解析server.xml是一种自动操作
            XML对象映射工具，它将创建server.xml中定义的对象
            实际容器的启动尚未开始。
        b5）将System.out和System.err分配给SystemLogHandler类
        b6）调用初始化所有组件，这使得每个对象自己注册
            JMX代理。
            在过程调用过程中，连接器也会初始化适配器。
            适配器是执行请求预处理的组件。
            典型的适配器是HTTP1.1（默认如果没有指定协议，
            org.apache.coyote.http11.Http11NioProtocol）
            AJP1.3 for mod_jk等

    c）Catalina.start（）
        c1）启动NamingContext并将所有JNDI引用绑定到它
        c2）启动<服务器>下的服务：
            StandardService  - >启动Engine（ContainerBase  - > Realm，Cluster等）
        c3）StandardHost（由服务启动）
                配置ErrorReportValvem以针对不同的HTTP执行适当的HTML输出
                错误代码
                启动管道中的阀门（至少是ErrorReportValve）
                配置StandardHostValve，
                    这个阀门将Webapp类加载器绑定到线程上下文
                    它也为请求找到会话
                    并调用上下文管道
                启动HostConfig组件
                    这个组件部署所有的webapps
                        （webapps＆conf / Catalina / localhost / *。xml）
                    HostConfig将为您的环境创建一个Digester，这个消化器
                    然后会调用ContextConfig.start（）
                        ContextConfig.start（）将处理默认的web.xml（conf / web.xml）
                        然后处理应用程序web.xml（WEB-INF / web.xml）

        c4）在容器的生命周期（StandardEngine）中有一个后台线程
            不断检查上下文是否已经改变。如果上下文发生变化（战争文件的时间戳，
            上下文xml文件，web.xml），然后发出重新加载（停止/删除/部署/启动）

    d）Tomcat通过HTTP端口接收请求
        d1）该请求由在ThreadPoolExecutor中等待的单独线程接收类。
             它正在等待常规ServerSocket.accept（）方法中的请求。
             当收到请求时，该线程唤醒。
        d2）ThreadPoolExecutor分配一个TaskThread来处理请求。
            它还提供了一个JMX对象名称到catalina容器（我相信没用过）
        d3）在这种情况下处理请求的处理器是Coyote Http11Processor，
            并调用处理方法。
            同一个处理器也在继续检查套接字的输入流
            直到达到保持活动点或连接断开。
        d4）使用内部缓冲区类（Http11InputBuffer）解析HTTP请求
            缓冲类解析请求行，标题等，并将结果存储在一个
            Coyote请求（不是HTTP请求）该请求包含所有HTTP信息，例如
            作为服务器名称，端口，方案等
        d5）处理器包含对适配器的引用，在这种情况下，它是
            CoyoteAdapter。一旦请求被解析，Http11Processor
            调用适配器上的service（）。在服务方法中，请求包含一个
            CoyoteRequest和CoyoteResponse（第一次为null）
            CoyoteRequest（Response）实现了HttpRequest（Response）和HttpServletRequest（Response）
            适配器解析并将所有内容与请求，cookie，上下文通过a进行关联
            Mapper等
        d6）解析完成后，CoyoteAdapter调用其容器（StandardEngine）
            并调用invoke（request，response）方法。
            这将从引擎级开始将HTTP请求发送到Catalina容器中
        d7）StandardEngine.invoke（）只是调用容器的pipeline.invoke（）
        d8）默认情况下，发动机只有一个阀门StandardEngineValve，这个阀门很简单
            调用主机流水线上的invoke（）方法（StandardHost.getPipeLine（））
        d9）StandardHost默认有两个阀门，即StandardHostValve和ErrorReportValve
        d10）标准主阀将正确的类装载机与当前螺纹相关联
             它还检索经理和与请求相关的会话（如果有的话）
             如果有会话访问（）被调用来保持会话存活
        d11）之后，StandardHostValve在关联的上下文上调用管道
             与请求。
        d12）由Context管道调用的第一个阀门是FormAuthenticator
             阀。然后StandardContextValve被调用。
             StandardContextValve调用与上下文关联的任何上下文侦听器。
             接下来它调用Wrapper组件上的管道（StandardWrapperValve）
        d13）在调用StandardWrapperValve期间，JSP包装器（Jasper）被调用
             这导致实际编译JSP。
             然后调用实际的servlet。
    e）调用servlet类
```

**图**

启动程序的UML序列图可以 [在这里找到。](http://tomcat.apache.org/tomcat-8.5-doc/architecture/startup/serverStartup.pdf)

**注释**

启动过程可以通过多种方式进行自定义，通过修改Tomcat代码和实现您自己的LifecycleListeners，然后在server.xml配置文件中注册。

