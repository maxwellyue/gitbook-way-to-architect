# 概述

#### Tomcat的组成 {#Terms}

1、Server

在Tomcat体系中， [服务器](http://tomcat.apache.org/tomcat-8.5-doc/config/server.html)（Server） 代表整个容器。Tomcat提供了用户很少会去定制的[服务器接口](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/Server.html)的默认实现 。

2、**Service**

[Service](http://tomcat.apache.org/tomcat-8.5-doc/config/service.html)是Server的内部组件，其中包含一个或多个连接器Connector以及一个引擎Engine。Service元素很少由用户自定义，因为默认实现很简单且足够：[Service interface](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/Service.html)。

3、**Engine**

一个[Engine](http://tomcat.apache.org/tomcat-8.5-doc/config/engine.html)表示一个特定的Service进行请求处理的流水线（pipeline）。由于Service可能有多个Connector，Engine会接收并处理来自这些连接器的所有请求，并将响应传递回适当的Connector以传输给客户端。 通过[Engine interface](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/Engine.html)可以实现提供自定义的Engine，但这种情况并不常见。

请注意，Engine可以通过jvmRoute参数用于Tomcat服务器集群。阅读群集文档以获取更多信息。

4、**Host**

[Host](http://tomcat.apache.org/tomcat-8.5-doc/config/host.html)会将网络的名称（例如www.yourcompany.com）关联到Tomcat服务器。一个Engine可能包含多个Host，Host元素也支持网络别名，例如yourcompany.com和abc.yourcompany.com。用户很少创建自定义[Hosts](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/Host.html)，因为 [StandardHost的实现](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/core/StandardHost.html)提供了重要的附加功能。

5、**Connector**

连接器处理与客户端的通信。Tomcat有多个连接器可用。其中包括用于大多数HTTP流量的[HTTP连接器](http://tomcat.apache.org/tomcat-8.5-doc/config/http.html)，尤其是在将Tomcat作为独立服务器运行时，以及[AJP连接器](http://tomcat.apache.org/tomcat-8.5-doc/config/ajp.html)，该[连接器](http://tomcat.apache.org/tomcat-8.5-doc/config/ajp.html)在将Tomcat连接到Web服务器（如Apache HTTPD服务器）时使用AJP协议。创建自定义连接器是一项重要的工作。

6、**Context**

一个[Context](http://tomcat.apache.org/tomcat-8.5-doc/config/context.html) 代表一个Web应用程序。Host可能包含多个上下文，每个上下文都有唯一的路径。通过[Context接口](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/Context.html)可以创建自定义的上下文，但这种情况很少见，因为[StandardContext](http://tomcat.apache.org/tomcat-8.5-doc/api/org/apache/catalina/core/StandardContext.html)提供了显著的附加功能。

#### 注释 {#Comments}

Tomcat旨在快速高效地实施Servlet规范。Tomcat成为本规范的参考实现，并且在遵守规范方面一直保持严格。与此同时，Tomcat的性能也受到了重视，现在它与其他servlet容器（包括商业容器）相当。

在最近的Tomcat版本中，主要从Tomcat 5开始，我们已经开始努力通过JMX使Tomcat的更多方面易于管理。此外，经理和管理网络应用程序已经得到了极大的增强和改进。随着产品成熟和规格变得更加稳定，可管理性成为我们关注的主要领域。

