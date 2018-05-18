Tomcat的用户一定都使用过其应用部署功能，无论是直接拷贝文件到webapps目录，还是修改server.xml以目录的形式部署，或者是增加虚拟主机，指定新的appBase等等。

但部署应用时，不知道你是否曾注意过这几点：

1. 如果在一个Tomcat内部署多个应用，甚至多个应用内使用了某个类似的几个不同版本，但它们之间却互不影响。这是如何做到的。

2. 如果多个应用都用到了某类似的相同版本，是否可以统一提供，不在各个应用内分别提供，占用内存呢。

3. 还有时候，在开发Web应用时，在pom.xml中添加了servlet-api的依赖，那实际应用的class加载时，会加载你的servlet-api 这个jar吗

以上提到的这几点，在Tomcat以及各类的应用服务器中，都是通过类加载器（**ClasssLoader**）来实现的。通过本文，你可以了解到Tomcat内部提供的各种类加载器，Web应用的class和资源等加载的方式，以及其内部的实现原理。在遇到类似问题时，更胸有成竹。

## **类加载器**

Java语言本身，以及现在其它的一些基于JVM之上的语言\(Groovy，Jython， Scala...\)，都是在将代码编译生成class文件，以实现跨多平台，write once, run anywhere。最终的这些class文件，在应用中，又被加载到JVM虚拟机中，开始工作。而把class文件加载到JVM的组件，就是我们所说的类加载器。而对于类加载器的抽象，能面对更多的class数据提供形式，例如网络、文件系统等。

Java中常见的那个**ClassNotFoundException**和**NoClassDefFoundError**就是类加载器告诉我们的。  


Servlet规范指出，容器用于加载Web应用内Servlet的class loader, 允许加载位于Web应用内的资源。但不允许重写java.\*, javax.\*以及容器实现的类。同时每个应用内使用**Thread.currentThread.getContextClassLoader\(\)**获得的类加载器，都是该应用区别于其它应用的类加载器等等。

  


根据Servlet规范，各个应用服务器厂商自行实现。所以像其他的一些应用服务器一样， Tomcat也**提供了多种的类加载器**，以便应用服务器内的class以及部署的Web应用类文件运行在容器中时，可以使用不同的class repositories。

  


在Java中，类加载器是以一种父子关系树来组织的。除Bootstrap外，都会包含一个parent 类加载器。\(这里写parent 类加载器，而不是父类加载器，不是为了装X，是为了避免和Java里的**父类**混淆\) **一般以类加载器需要加载一个class或者资源文件的时候，他会先委托给他的parent类加载器，让parent类加载器先来加载，如果没有，才再在自己的路径上加载**。这就是人们常说的双亲委托，即把类加载的请求委托给parent。

但是...，这里需要注意一下

> 对于Web应用的类加载，和上面的双亲委托是有区别的。

在Tomcat中，涉及到的类加载器大致有以下几类，像官方文档里这张表示一样，这里对于Bootstrap和System这种加载Java基础类的我们不做分析，主要来看一下后面的Common和WebappX这两类class loader。

  


```
     Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ...

```

  


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
>
> ```

  


其中initClassLoaders方法，会根据catalina.properties的配置，创建相应的classloader。由于默认只配置了common.loader属性，所以其中只会创建一个出来

  


> ```
> private
> void
> initClassLoaders
> ()
> {
> try
> {
> commonLoader
> =
> createClassLoader
> (
> "common"
> ,
> null
> );
> if
> (
> commonLoader
> ==
> null
> )
> {
> // no config file, default to this loader - we might be in a 'single' env.
> commonLoader
> =
> this
> .
> getClass
> ().
> getClassLoader
> ();
> }
> catalinaLoader
> =
> createClassLoader
> (
> "server"
> ,
> commonLoader
> );
> sharedLoader
> =
> createClassLoader
> (
> "shared"
> ,
> commonLoader
> );
> }
> catch
> (
> Throwable
> t
> )
> {
> handleThrowable
> (
> t
> );
> log
> .
> error
> (
> "Class loader creation threw exception"
> ,
> t
> );
> System
> .
> exit
> (
> 1
> );
> }
> }
> ```

  


所以，后面线程中绑定的都一直是commonClassLoader。

  


然后，当一个应用启动的时候，会为其创建对应的WebappClassLoader。此时会将commonClassLoader设置为其parent。下面的代码是StandardContext类在启动时创建WebappLoader的代码

  


> ```
> if
> (
> getLoader
> ()
> ==
> null
> )
> {
> WebappLoader
> webappLoader
> =
> new
> WebappLoader
> (
> getParentClassLoader
> ());
> webappLoader
> .
> setDelegate
> (
> getDelegate
> ());
> setLoader
> (
> webappLoader
> );
> }
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
> private WebappClassLoaderBase createClassLoader()
> throws Exception {
>
>     Class
> <
> ?
> >
>  clazz = Class.forName(loaderClass);
>     WebappClassLoaderBase classLoader = null;
>
> if (parentClassLoader == null) {
> parentClassLoader = context.getParentClassLoader();
>     }
>     Class
> <
> ?
> >
> [] argTypes = { ClassLoader.class };
>     Object[] args = { parentClassLoader };
>     Constructor
> <
> ?
> >
>  constr = clazz.getConstructor(argTypes);
>     classLoader = (WebappClassLoaderBase) constr.newInstance(args);
> return classLoader;
> }
>
> ```

  


配置等信息使用前面Loader内的配置。

应用的classLoader也配置好之后，我们再来看真正应用需要class的时候，是如何子优先的。

在loadClass的时候，会调用到WebappClassLoader的loadClass方法，此时，查找一个class的步骤总结这样几步：  


这里把方法中分步的注释拿来罗列一下，

  


> 1.  \(0\) Check our previously loaded local class cache
>
> 2.  \(0.1\) Check our previously loaded class cache
>
> 3.  \(0.2\) Try loading the class with the system class loader, to prevent  
>
>
>     the webapp from overriding Java SE classes. This implements SRV.10.7.2
>
> 4. 然后，会判断是否启用了securityManager，启用时会进行packageAccess的检查。

  


主要判断已加载的类里是否已经包含，然后避免Java SE的classes被覆盖，packageAccess的检查。

  


之后，开始了我们的父优先子优先的流程。这里判断是否使用delegate时，对于一些容器提供的class，也会跳过。

  


> boolean delegateLoad = delegate \|\| **filter**\(name\);

  


这里的filter就用来过滤容器提供的类以及servlet-api的类。

  


> ```
> protected
> synchronized
> boolean
> filter
> (
> String
> name
> )
> {
> if
> (
> name
> ==
> null
> )
> return
> false
> ;
> // Looking up the package
> String
> packageName
> =
> null
> ;
> int
> pos
> =
> name
> .
> lastIndexOf
> (
> '.'
> );
> if
> (
> pos
> !=
> -
> 1
> )
> packageName
> =
> name
> .
> substring
> (
> 0
> ,
> pos
> );
> else
> return
> false
> ;
> packageTriggersPermit
> .
> reset
> (
> packageName
> );
> if
> (
> packageTriggersPermit
> .
> lookingAt
> ())
> {
> return
> false
> ;
> }
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
>
> ```

  


此时如果没找到，就走到下面的代码，开始查找本地的资源库\(repository\)和子优先时一样：

> ```
> // (2) Search local repositories
> if
> (
> log
> .
> isDebugEnabled
> ())
> log
> .
> debug
> (
> "  Searching local repositories"
> );
> try
> {
> clazz
> =
> findClass
> (
> name
> );
> if
> (
> clazz
> !=
> null
> )
> {
> if
> (
> log
> .
> isDebugEnabled
> ())
> log
> .
> debug
> (
> "  Loading class from local repository"
> );
> if
> (
> resolve
> )
> resolveClass
> (
> clazz
> );
> return
> (
> clazz
> );
> }
> }
> catch
> (
> ClassNotFoundException
> e
> )
> {
> // Ignore
> }
> ```

  


如果父优先和子优先都没能查找到需要的class，此时会抛出  


> ```
> throw new ClassNotFoundException(name);
>
> ```

关于上面代码，有一个地方，感兴趣的同学可以再深入了解下，

> ```
> clazz = Class.forName(name, false, parent);
>
> ```

  


也许你这么多年一直直接用Class.forName，没管过后面还可以多传两个参数。

关于ClassLoader，你还想了解什么？

  
Tomcat类加载器及应用间class隔离与共享

  


