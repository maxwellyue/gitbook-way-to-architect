JDK的类加载机制是双亲委派：先委托给他的parent类加载器，让parent类加载器先来加载，如果没有，才再在自己的路径上加载。

但是Tomcat的类加载机制与此不同。

Tomcat在初始化时，会创建以下4种类加载器（[Tomcat 8.5](http://tomcat.apache.org/tomcat-8.5-doc/class-loader-howto.html)）。

```
       Bootstrap           
          |
       System              
          |
       Common          
       /     \
  Webapp1   Webapp2 ...    
```

1. Bootstrap：负责加载JVM提供的基础的运行时类（即rt.jar）以及${JAVA\_HOME}/jre/lib/ext下的类（相当于JDK本身的Bootstarp和Extention这两个类加载器的功能）。
2. System：该类加载器通常会加载环境变量CLASSPATH中的类，但是Tomcat并不是如此，它会忽略CLASSPATH，而去加载以下jar文件：
①



## 

## 

## 

## 

## 

## **Webapp类加载器**

正如上面内容所说，Webapp类加载器，相对于传统的Java的类加载器，最主要的区别是

> **子优先**\(child first\)

也就是说，在Web应用内，需要加载一个类的时候，不是先委托给parent，而是先自己加载，在自己的类路径上找不到才会再委托parent。

> 但是此处的子优先有些地方需要注意的是，Java的基础类不允许其重新加载，以及servlet-api也不允许重新加载。

那为什么要先child之后再parent呢？我们前面说是Servlet规范规定的。但确实也是实际需要。假如我们两个应用内使用了相同命名空间里的一个class，一个使用的是Spring 2.x，一个使用的是Spring 3.x。如果是parent先加载的话，在第一个应用加载后，第二个应用再需要的时候，就直接从parent里拿到了，但是却不符合需要。

另外一点是，各个Web应用的类加载器，是相互独立的，即WebappClassloader的多个实例，只有这样，多个应用之间才可能使用不同版本的相同命令空间下的类库，而不互相受影响。

该类加载器会加载Web应用的WEB-INF/classes内的class和资源文件，以及WEB-INF/lib下的所有jar文件。

当然，有些时候，有需要还按照传统的Java类加载器加载class时，Tomcat内提供了配置，可以实现父优先。

## **Common 类加载器**

通过上面的class loader组织的图，可以知道Common 类加载器，是做为webapp类加载器的parent存在的。它是在以下文件中进行配置的：

> **TOMCAT\_HOME/conf/catalina.properties**

文档中给的样例：

对于目录结尾的，视为class文件的加载路径，对于目录/\*.jar结尾的，则视为目录下所有jar会被加载。

这个配置，默认已经包含了Tomcat的base下的lib目录和home下的lib目录。\(关于catalina.base这个，可以看之前写IDE内Tomcat工作原理的文章[你一定不知道IDE里的Tomcat是怎么工作的！](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI3MTEwODc5Ng%3D%3D%26mid%3D401107149%26idx%3D1%26sn%3D908bd8ba76b38417570056795626c163%26scene%3D21%23wechat_redirect)\)

> **common.loader**="${catalina.base}/lib","${catalina.base}/lib/\*.jar","${catalina.home}/lib","${catalina.home}/lib/\*.jar"

所以，lib目录下的class和jar文件，在启动时就都被加载了。

一般来说，这个类加载器用来加载一些既需要Tomcat容器内和所有应用共同可见的class，应用的class不建议放到这儿加载。

介绍完这两个加载器之后，我们来看文章开始时提到的几个问题：

* 多个应用之间类库不互相冲突，是由于使用了不同的类加载器进行加载的。彼此之间如同路人。即使看起来同样一个类，使用不同的类加载器加载，也是不同的对象，这点要引起注意。

* 多个应用之间，如果大家使用了相同的类库，而且数据众多，为了避免重复加载占用内存，就可以用到我们的Common 类加载器。只要在配置中指定对应的目录，然后提取出共用的文件即可。我在之前的公司开发应用服务器时，就有客户有这样的需求。

* 对于我们应用内提供的Servlet-api，其实应用服务器是不会加载的，因为容器已经自已加载过了。当然，这里不是因为父优先还是子优先的问题，而是这类内容，是不允许被重写的。如果你应用内有一个叫javax.servlet.Servlet的class，那加载后可能就影响了应用内的正常运行了。

我们看在Tomcat6.x中加载一个包含servlet 3.x api的jar，会直接提示jar not loaded.

**类加载器实现分析**

在Tomcat启动时，会创建一系列的类加载器，在其主类Bootstrap的初始化过程中，会先初始化classloader，然后将其绑定到Thread中。

> ```
> public void init() throws Exception {
>
>     initClassLoaders();
>
> Thread.currentThread().setContextClassLoader(catalinaLoader);
>
>     SecurityClassLoad.securityClassLoad(catalinaLoader);  }
> ```

其中initClassLoaders方法，会根据catalina.properties的配置，创建相应的classloader。由于默认只配置了common.loader属性，所以其中只会创建一个出来

> ```
>
> ```

所以，后面线程中绑定的都一直是commonClassLoader。

然后，当一个应用启动的时候，会为其创建对应的WebappClassLoader。此时会将commonClassLoader设置为其parent。下面的代码是StandardContext类在启动时创建WebappLoader的代码

> ```
>
> ```

这里的getParentClassLoader会从当前组件的classLoader一直向上，找parent classLoader设置。之后注意下一行代码

> webappLoader.**setDelegate**

这就是在设置后面Web应用的类查找时是父优先还是子优先。这个配置可以在server.xml里，对Context组件进行配置。

即在Context元素下可以嵌套一个**Loader**元素，配置Loader的delegate即可，其默认为false，即子优先。类似于这样

> &lt;Context&gt;
>
> &lt;Loader className="" **delegate**="true"/&gt;
>
> &lt;/Context&gt;

注意Loader还有一个属性是reloadable，用于表明对于/WEB-INF/classes/ 和 /WEB-INF/lib 下资源发生变化时，是否重新加载应用。这个特性在开发的时候，还是很有用的。

如果你的应用并没有配置这个属性，想要重新加载一个应用，只需要使用manager里的reload功能就可以。

有点跑题，回到我们说的delgate上面来，配置之后，可以指定Web应用类加载时，到底是使用父优先还是子优先。

这里的WebappLoader，就开始了正式的创建WebappClassLoader

> ```
> 配置等信息使用前面Loader内的配置。
> ```

应用的classLoader也配置好之后，我们再来看真正应用需要class的时候，是如何子优先的。

在loadClass的时候，会调用到WebappClassLoader的loadClass方法，此时，查找一个class的步骤总结这样几步：

这里把方法中分步的注释拿来罗列一下，

> 1. \(0\) Check our previously loaded local class cache
>
> 2. \(0.1\) Check our previously loaded class cache
>
> 3. \(0.2\) Try loading the class with the system class loader, to prevent
>
> ```
> the webapp from overriding Java SE classes. This implements SRV.10.7.2
> ```
>
> 1. 然后，会判断是否启用了securityManager，启用时会进行packageAccess的检查。

主要判断已加载的类里是否已经包含，然后避免Java SE的classes被覆盖，packageAccess的检查。

之后，开始了我们的父优先子优先的流程。这里判断是否使用delegate时，对于一些容器提供的class，也会跳过。

> boolean delegateLoad = delegate \|\| **filter**\(name\);

这里的filter就用来过滤容器提供的类以及servlet-api的类。

> ```
>
> ```

然后确定到底是父优先，还是子优先，开始类的加载

**父优先**

> ```
> // (1) Delegate to our parent if requested
> if (delegateLoad) {
> if (log.isDebugEnabled())
> log.debug("  Delegating to parent classloader1 " + parent);
> try {
>         clazz = Class.forName(name, false, parent);
> if (clazz != null) {
> if (log.isDebugEnabled())
> log.debug("  Loading class from parent");
> if (resolve)
>                 resolveClass(clazz);
> return (clazz);
>         }
>     } catch (ClassNotFoundException e) {
> // Ignore
>     }
> }
> ```

此时如果没找到，就走到下面的代码，开始查找本地的资源库\(repository\)和子优先时一样：

> ```
> 如果父优先和子优先都没能查找到需要的class，此时会抛出
> ```

> ```
> throw new ClassNotFoundException(name);
> ```

关于上面代码，有一个地方，感兴趣的同学可以再深入了解下，

> ```
> clazz = Class.forName(name, false, parent);
> ```

也许你这么多年一直直接用Class.forName，没管过后面还可以多传两个参数。

关于ClassLoader，你还想了解什么？

Tomcat类加载器及应用间class隔离与共享

