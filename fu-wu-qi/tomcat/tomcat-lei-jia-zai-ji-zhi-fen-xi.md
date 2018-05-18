# Tomcat类加载机制分析

**JDK的类加载机制是双亲委派**：先委托给他的parent类加载器，让parent类加载器先来加载，如果没有，才再在自己的路径上加载。

**但是Tomcat的类加载机制与此不同。**

Tomcat在初始化时，会创建以下4种类加载器（[Tomcat 8.5](http://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)）。

```text
       Bootstrap           
          |
       System              
          |
       Common          
       /     \
  Webapp1   Webapp2 ...
```

* **Bootstrap**：负责加载JVM提供的基础的运行时类（即`rt.jar`）以及`${JAVA\_HOME}/jre/lib/ext`下的类（相当于JDK本身的`Bootstrap`和`Extension`这两个类加载器的功能）。
* **System**：该类加载器通常会加载环境变量CLASSPATH中的类，但是Tomcat并不是如此，它会忽略CLASSPATH，而去加载`$CATALINA_HOME/bin`目录下的3个jar（启动脚本中写死的）：
  * `bootstrap.jar`
  * `tomcat-juli.jar`（如果`$CATALINA_BASE/bin`目录下也有这个包，会使用`$CATALINA_BASE/bin`目录下的）
  * `commons-daemon.jar` 
* **Common** ：该类加载器加载的类，对Tomcat自身和所有Web应用都可见。通常情况下，应用的类文件不应该放在Common ClassLoader中。Common会扫描 `$CATALINA_BASE/conf/catalina.properties`中common.loader属性指定的路径中的类文件。默认情况下，它会按顺序去下列路径中去加载：
  * `$CATALINA_BASE/lib`中未打包的classes和resources
  * `$CATALINA_BASE/lib`中的jar文件
  * `$CATALINA_HOME/lib`中未打包的classes和resources
  * `$CATALINA_HOME/lib`中的jar文件

> 默认情况下，这些路径下的jar主要有以下这些：
>
> * annotations-api.jar — JavaEE annotations classes.
> * catalina.jar — Implementation of the Catalina servlet container portion of Tomcat.
> * catalina-ant.jar — Tomcat Catalina Ant tasks.
> * catalina-ha.jar — High availability package.
> * catalina-storeconfig.jar — Generation of XML configuration files from current state
> * catalina-tribes.jar — Group communication package.
> * ecj-\*.jar — Eclipse JDT Java compiler.
> * el-api.jar — EL 3.0 API.
> * jasper.jar — Tomcat Jasper JSP Compiler and Runtime.
> * jasper-el.jar — Tomcat Jasper EL implementation.
> * jsp-api.jar — JSP 2.3 API.
> * servlet-api.jar — Servlet 3.1 API.
> * tomcat-api.jar — Several interfaces defined by Tomcat.
> * tomcat-coyote.jar — Tomcat connectors and utility classes.
> * tomcat-dbcp.jar — Database connection pool implementation based on package-renamed copy of Apache Commons Pool and Apache Commons DBCP.
> * tomcat-i18n-\*\*.jar — Optional JARs containing resource bundles for other languages. As default bundles are also included in each individual JAR, they can be safely removed if no internationalization of messages is needed.
> * tomcat-jdbc.jar — An alternative database connection pool implementation, known as Tomcat JDBC pool. See documentation for more details.
> * tomcat-util.jar — Common classes used by various components of Apache Tomcat.
> * tomcat-websocket.jar — WebSocket 1.1 implementation
> * websocket-api.jar — WebSocket 1.1 API

* **WebappX** 每一个部署在Tomcat中的web应用，Tomcat都会为其创建一个WebappClassloader，它会去加载应用`WEB-INF/classes`目录下所有未打包的classes和resources，然后再去加载`WEB-INF/lib`目录下的所有jar文件。每个应用的WebappClassloader都不同，因此，它加载的类只对本应用可见，其他应用不可见（这是实现web应用隔离的关键）。

正如上面提到的，WebappClassloader的行为与Java默认的双亲委派模型是有所区别的。当WebappClassloader收到加载类的请求时，它**首先**在自己的路径（repository）中去寻找类的class文件（而不是委托给父类去加载）。但是有例外：JRE中的类库是不允许重写的！

**从应用的视角来看，当有类加载的请求时，class或者resource的查找顺序是这样的：** 

 1. JVM中的类库，如`rt.jar`和`$JAVA_HOME/jre/lib/ext`目录下的jar

 2. 应用的`/WEB-INF/classes`目录 

3. 应用的`/WEB-INF/lib/*.jar` 

4. `SystemClassloader`加载的类（如上所述） 

5. `CommonClassloader`加载的类（如上所述）

**但是，如果你的应用配置了**`<loader delegate="true"/>`**，那么查找顺序就会变为：** 

 1. JVM中的类库，如`rt.jar`和`$JAVA_HOME/jre/lib/ext`目录下的jar 

2. `SystemClassloader`加载的类（如上所述） 

3. `CommonClassloader`加载的类（如上所述） 

4. 应用的`/WEB-INF/classes`目录 

5. 应用的`/WEB-INF/lib/*.jar`

## QA

Q：默认配置下，在项目中如果在一个Tomcat内部署多个应用，甚至多个应用内使用了某个类似的几个不同版本，但它们之间却互不影响。这是如何做到的？

> A：多个应用之间的相同类库，由于使用不同的类加载器进行加载，它们仍然是不同的对象。  
>  对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性：比较两个类是否“相等”，只有在这两个类是由同一个类加载器的前提下才有意义，否则即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。这里所指的相等，包括代表类的Class类的对象的equals\(\)方法、isAssignableFrom\(\)方法、isInstance\(\)方法的返回结果，也包括使用instanceof关键字做对象所属关系判定等情况。

Q：如果多个应用都用到了某类似的相同版本，是否可以统一提供，不在各个应用内分别提供，占用内存？

> A：可以，可以将这些公共的类库放在Common类加载器的扫描范围，具体做法是在`$CATALINA_BASE/conf/catalina.properties`中`common.loader`属性中追加你自己的应用共用的jar。`common.loader`默认值为：
>
> ```text
> common.loader="${catalina.base}/lib",
> "${catalina.base}/lib/*.jar",
> "${catalina.home}/lib",
> "${catalina.home}/lib/*.jar"
> ```

Q：在开发Web应用时，在pom.xml中添加了servlet-api的依赖，那实际应用的class加载时，会加载应用本身引入的servlet-api这个jar吗？

> A：不会。对于我们应用内提供的Servlet-api，应用服务器是不会加载的，因为容器已经自已加载过了。这里不是因为父优先还是子优先的问题，而是这类内容，是不允许被重写的。如果你应用内有一个叫javax.servlet.Servlet的class，那加载后可能就影响了应用内的正常运行了。

Q：假如有一个jar包（非Java规范或Servlet规范的实现包，只是普通的其他第三方包），这个jar文件在`$CATALINA_HOME/bin`（或者在`$CATALINA_HOME/lib`）目录、应用本身的/WEB-INF/lib目录下都各有一份，那么最终应用中使用的是哪个jar？

> A：分为两种情况讨论
>
> *      1.**Jar在**`$CATALINA_HOME/bin`**和**`WEB-INF/lib`**下** 这种情况下，总是以应用中的jar为准：因为自己放在`$CATALINA_HOME/bin`下的jar根本不会被加载，SystemClassloader只会加载`$CATALINE_HOME/bin`下的自带的3个jar（上面提高的那三个），这是在Tomcat中的启动脚本中`catalina.sh`写死的，其他放在该目录的jar，不会被加载。
> *        2.**Jar在**`$CATALINA_HOME/lib`**和**`WEB-INF/lib`**下**  
>   这种情况下，又有两种情形：①默认情形`(<loader delegate="false">)`，按照Tomcat的类加载机制，WebappClassloader加载，而WebappClassLoader首先在自己的路径范围（即`/WEB-INF/lib`目录）查找，发现自己的路径内含有要加载的类的class文件，就会将其load到JVM中，结束这次类加载；②修改了`Tomcat`默认的类加载机制，即配置了`<loader delegate=true>`，那么`WebappClassloader`在收到加载jar内某各类的加载请求时，会直接将类加载请求委托给父类`CommonClassloader`，`CommonClassloader`就会去自己的路径范围去查找对应class文件，完成加载。  
>
>
>   **验证思路**：
>
>   ①写一个类Dog，打包，将其放在`$CATALINA_HOME/bin`或者`$CATALINA_HOME/lib`下
>
>   ```text
>   public class Dog{
>   public Dog(){
>      System.out.println("a dog born in tomcat internal");
>   }
>   }
>   ```
>
>   ②将Dog类该写为，打包，新增webapp项目，将jar放在该webapp项目的`/WEB-INF/lib`目录下
>
>   ```text
>   public class Dog{
>   public Dog(){
>      System.out.println("a dog born in webapp");
>   }
>   }
>   ```
>
>   ③使该项目在该Tomcat下运行，观察输出内容。  
>
>
>   ④修改`$CATALINA_HOME/conf/Context.xml`，添加内容`<Loader delegate="true"/>`，再次运行项目，观察输出内容



**TODO**：这种类加载机制的实现细节。比如，当java.util.ArralyList的加载时的流程走向的实现细节。

## 参考

[Tomcat-8.5-官方文档：Class Loader HOW-TO](http://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)

 [Tomcat类加载器及应用间class隔离与共享 ](https://zhuanlan.zhihu.com/p/24168200)

