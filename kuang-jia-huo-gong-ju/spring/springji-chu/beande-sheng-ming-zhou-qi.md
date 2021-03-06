# Xml格式的应用启动

---

Xml和注解是我们进行使用Spring进行开发时用到的两种主要配置方式。本文简单介绍Xml格式的应用的启动流程。

从启动应用的代码开始入手：

```java
public class Launcher {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring/spring.xml");
        CountDownLatch latch = new CountDownLatch(1);
        try {
            latch.await();
        } catch (InterruptedException ignored) {
        }
    }
}
```

这里，只是创建了一个ClassPathXmlApplicationContext的实例，并传入了一个字符串“classpath:spring/spring.xml”，来看它的构造器：

```java
//上面使用的构造器
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[] {configLocation}, true, null);
}
//最终使用的构造器
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}
```

因为parent为null，所以这里先忽略super\(parent\)；主要做了两件事：setConfigLocations\(configLocations\)和refresh\(\);

setConfigLocations\(configLocations\)是在其父类AbstractRefreshableConfigApplicationContext中：

```java
public void setConfigLocations(@Nullable String... locations) {
    if (locations != null) {
        this.configLocations = new String[locations.length];
        for (int i = 0; i < locations.length; i++) {
        this.configLocations[i] = resolvePath(locations[i]).trim();
        }
    }else {
        this.configLocations = null;
    }
}
//resolvePath方法：如果给的路径中含有占位符${...}，则使用环境变量来替换掉
protected String resolvePath(String path) {
    return getEnvironment().resolveRequiredPlaceholders(path);
}
```

其实this.configLocations最终就是我们传入的那个字符串，即：

```java
this.configLocations=["classpath:spring/spring.xml"]
```

到这里，setConfigLocations\(configLocations\)就结束了，该方法就是将我们传入的一个表示路径的字符串设置给ClassPathXmlApplicationContext。

再来看refesh\(\)，该方法是接口ConfigurableApplicationContext中定义的方法，是应用真正启动的方法，所以这个方法非常重要。

具体到ClassPathXmlApplicationContext，这个refesh\(\)的实现是在其父类AbstractApplicationContext中：

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
    // Prepare this context for refreshing.
    prepareRefresh();

    // Tell the subclass to refresh the internal bean factory.
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

    // Prepare the bean factory for use in this context.
    prepareBeanFactory(beanFactory);

    try {
        // Allows post-processing of the bean factory in context subclasses.
        postProcessBeanFactory(beanFactory);

        // Invoke factory processors registered as beans in the context.
        invokeBeanFactoryPostProcessors(beanFactory);

        // Register bean processors that intercept bean creation.
        registerBeanPostProcessors(beanFactory);

        // Initialize message source for this context.
        initMessageSource();

        // Initialize event multicaster for this context.
        initApplicationEventMulticaster();

        // Initialize other special beans in specific context subclasses.
        onRefresh();

        // Check for listener beans and register them.
        registerListeners();

        // Instantiate all remaining (non-lazy-init) singletons.
        finishBeanFactoryInitialization(beanFactory);

        // Last step: publish corresponding event.
        finishRefresh();
    }

    catch (BeansException ex) {
        // Destroy already created singletons to avoid dangling resources.
        destroyBeans();
        // Reset 'active' flag.
        cancelRefresh(ex);
        // Propagate exception to caller.
        throw ex;
    }

    finally {
        // Reset common introspection caches in Spring's core, since we
        // might not ever need metadata for singleton beans anymore...
        resetCommonCaches();
    }
    }
}
```

 忽略同步措施，只看具体逻辑。Spring对refresh\(\)方法注释的非常详细，鉴于翻译可能表达的不太准备，保留了原文注释。我们将该方法内的每个方法看做一个步骤进行追踪。

**第一步：prepareRefresh\(\)，刷新该context的准备工作。**

```java
protected void prepareRefresh() {
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isInfoEnabled()) {
        logger.info("Refreshing " + this);
    }

    // Initialize any placeholder property sources in the context environment
    initPropertySources();

    // Validate that all properties marked as required are resolvable
    // see ConfigurablePropertyResolver#setRequiredProperties
    getEnvironment().validateRequiredProperties();

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

AbstractApplicationContext中定义了几个变量表示当前Context的状态：

```java
/** 当前context启动的时间，毫秒 */
private long startupDate;

/** 当前context是否被激活 */
private final AtomicBoolean active = new AtomicBoolean();

/** 当前context是否已经被关闭 */
private final AtomicBoolean closed = new AtomicBoolean();
```

 所以，prepareRefresh\(\)中首先记下当前context启动的时间，并设置该context已激活，且没有被关闭。

initPropertySources翻译过来就是初始化配置资源（这里的property是表示配置项而非属性），是交给子类选择性实现，具体到ClassPathXmlApplicationContext，它没有实现该方法。这其实是Spring留给用户进行个性化的配置来源的，比如应用中有一些特殊配置项或者要求，可以在这里将这些特殊的配置项设置到该Context的Environment中，比如：

```java
public class MyClassPathXmlApplicationContext extends ClassPathXmlApplicationContext {
    @Override
    protected void initPropertySources() {
        //要求必须含有该var这个配置
        getEnvironment().setRequiredProperties("var");
    }
}
```

getEnvironment\(\).validateRequiredProperties\(\)：先看getEnvironment\(\)

```java
//getEnvironment()的时候，先看该context中有没有environment，没有的话就新建
public ConfigurableEnvironment getEnvironment() {
    if (this.environment == null) {
        this.environment = createEnvironment();
    }
    return this.environment;
}
//创建一个StandardEnvironment
protected ConfigurableEnvironment createEnvironment() {
    return new StandardEnvironment();
}
```

到这里，该context已经有环境environment了。校验的逻辑是在StandardEnvironment的父类AbstractEnvironment中，且又委托给了配置解析器PropertyResolver：

```java
private final MutablePropertySources propertySources = new MutablePropertySources(this.logger);

private final ConfigurablePropertyResolver propertyResolver = new PropertySourcesPropertyResolver(this.propertySources);

... ... 
public void validateRequiredProperties() throws MissingRequiredPropertiesException {
    this.propertyResolver.validateRequiredProperties();
}
```

由于这里并没有设置配置项为必须（setRequiredProperties\("property"\)），所以validateRequiredProperties相当于没有什么必须的配置项进行校验。

所以，第一步的准备工作prepareRefresh\(\)主要是：设置Context的启动时间/状态，创建Context的Environment，校验必须配置项是否能解析到值。

 

