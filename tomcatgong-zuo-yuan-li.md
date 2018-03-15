TODO 有待系统梳理

**Server**：指的就是整个 Tomcat 服 务器，包含多组服务，负责管理和 启动各个 Service，同时监听 8005 端口发过来的 shutdown 命令，用 于关闭整个容器 ；

**Service**：Tomcat 封装的、对外提 供完整的、基于组件的 web 服务， 包含 Connectors、Container 两个 核心组件，以及多个功能组件，各 个 Service 之间是独立的，但是共享 同一 JVM 的资源 ；

**Connector**：Tomcat 与外部世界的连接器，监听固定端口接收外部请求，传递给 Container，并 将 Container 处理的结果返回给外部；

**Container**：Catalina，Servlet 容器，内部有多层容器组成，用于管理 Servlet 生命周期，调用 servlet 相关方法。

**Loader**：封装了 Java ClassLoader，用于 Container 加载类文件； Realm：Tomcat 中为 web 应用程序提供访问认证和角色管理的机制；

**JMX**：Java SE 中定义技术规范，是一个为应用程序、设备、系统等植入管理功能的框架，通过 JMX 可以远程监控 Tomcat 的运行状态；

**Jasper**：Tomcat 的 Jsp 解析引擎，用于将 Jsp 转换成 Java 文件，并编译成 class 文件。 Session：负责管理和创建 session，以及 Session 的持久化\(可自定义\)，支持 session 的集  
群。

**Pipeline**：在容器中充当管道的作用，管道中可以设置各种 valve\(阀门\)，请求和响应在经由管 道中各个阀门处理，提供了一种灵活可配置的处理请求和响应的机制。

**Naming**：命名服务，JNDI， Java 命名和目录接口，是一组在 Java 应用中访问命名和目录服务的 API。命名服务将名称和对象联系起来，使得我们可以用名称访问对象，目录服务也是一种命名 服务，对象不但有名称，还有属性。Tomcat 中可以使用 JNDI 定义数据源、配置信息，用于开发 与部署的分离。

  




  


[Tomcat组成与工作原理](https://juejin.im/post/58eb5fdda0bb9f00692a78fc)

  




