# Spring整体架构

---

Spring是一个分层架构，大约分为20几个模块，且随着版本更新，越来越多的模块被包含进来，以下是Spring 5.0.0RC2版本的整体架构图。



**核心容器：Core Container**

CoreContainer是Spring实现Ioc容器的核心，由Core，Beans，Context和Expression Language模块组成。

* **Core**：spring框架的基本核心工具类，是其他组件的基本核心。

* **Bean**：访问配置文件，创建和管理bean。

* **Context**：提供对象的访问方法以及对Spring核心的扩展，如对国际化，事件传播，资源加载等。

* **Expression Language**：提供强大的表达式语言，用于在运行时查询和操作对象。

**数据访问和整合：Data Access/Integration**

任何应用都离不开数据，Data Access/Integration是Spring为应用访问数据提供的封装，主要由JDBC，ORM，OXM，JMS和Transaction模块构成。

* **JDBC**：对JDBC数据访问进行封装的所有类。

* **ORM**：对流行的对象关系映射API\(Hibernate，MyBatis， JPA等\)提供一个交互层。

* **Transaction**：事务管理  ，支持编程和声明式两种形式。

* **JMS**：对生产和消费消息的抽象。

* **OXM**：对Object/XMl映射实现的抽象层。

**WEB**

为基于Web的应用程序提供上下文。

* Web：提供了基础的面向Web的特性，如多文件上传，使用Servlet Listeners初始化Ioc容器以及一个面向web的应用上下文。

* Web-Servlet：提供了Spring的MVC的实现。

* Web-Socket：提供了对Web Socket的支持。

* Web-Porlet：提供了用于Portlet环境和Web-Servlet模块的MVC实现  。

**AOP**

AOP模块是面向切面编程的实现。

* AOP：提供了符合AOP标准的面向切面编程的实现。
* Aspects：提供了对AspectJ的集成支持。
* Instrucmentation：提供在特定的应用程序服务器中类级别修改（修改字节码等）支持和类加载器的实现。

**Test**

Test模块支持使用JUnit和TestNG等对Spring组件进行测试。

**Messaging**

实现基于消息（message）的应用程序开发。

随着Spring的发展，可能会有越来越多的模块被加入，也会有一些被淘汰的技术（如Structs）从Spring中剔除，但是对于容器以及AOP的实现，基本不会有变化，这些内容是Spring的核心功能，了解它们对于如何更好地使用Spring有很大帮助。

# 参考

---

[Overview of Spring Framework](https://docs.spring.io/spring/docs/5.0.0.RC2/spring-framework-reference/overview.html)

《Spring源码深度解析》

  


