# 请求处理流程

#### 请求处理流程 {#Request_Process_Flow}

本节描述Tomcat用来处理传入请求的过程。这个过程在很大程度上由Servlet规范定义，该规范概述了必须发生的事件的顺序。

**描述**

TODO

**图**

请求过程的UML序列图可以 [在这里找到。](http://tomcat.apache.org/tomcat-8.5-doc/architecture/requestProcess/request-process.png)

[这里](http://tomcat.apache.org/tomcat-8.5-doc/architecture/requestProcess/authentication-process.png) 提供了认证过程的UML序列图。

**注释**

在请求到达处理它的servlet之前，Servlet规范提供了很多机会来侦听（使用侦听器）或修改（使用过滤器）请求处理过程。  


