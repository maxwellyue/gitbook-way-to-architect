## **BeanPostProcessor**

---

BeanPostProcessor，字面意思就是“Bean之后的处理器”（post可以理解为after），接口如下：

* `Object postProcessBeforeInitialization(Object bean, String beanName)`：该方法会在与beanName对应的Bean完成初始化（且已经被填充了属性值）之后，但在其他初始化回调（如该Bean实现了InitializingBean的afterPropertiesSet或者该Bean定制了init-method）之前，进行调用。返回值bean可以是原始Bean（即入参Object bean）的包装类。

* `Object postProcessAfterInitialization(Object bean, String beanName)`：该方法会在与beanName对应的Bean完成初始化（且已经被填充了属性值）之后，且在其他初始化回调（如该Bean实现了InitializingBean的afterPropertiesSet或者该Bean定制了init-method）之后，进行调用。返回值bean可以是原始Bean（即入参Object bean）的包装类。

即BeanPostProcessor可以在Bean完成初始化（且已填充完属性）时，对Bean做一些处理。![](/assets/屏幕快照 2018-10-27 下午6.01.55.png)**InstantiationAwareBeanPostProcessor**

它定义了比BeanPostProcessor的postProcessBeforeInitialization和postProcessAfterInitialization更早的方法：

```java
//在实例化之前的回调（即在new之前），返回值往往是原始对象的包装
Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName)
//在实例化之后，但是还没有填充属性（即在new之后，但set属性之前）
postProcessAfterInstantiation(Object bean, String beanName)
//填充属性时的回调方法(即在set属性时)
PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName)
```

就英文单词字面意思来看，Instantiation表示实例化，Initialization表示初始化，这5个方法的执行的顺序为\(越靠前越先执行\)：postProcessBeforeInstantiation &lt; postProcessAfterInstantiation &lt; postProcessPropertyValues &lt; postProcessBeforeInitialization &lt; postProcessAfterInitialization。

**SmartInstantiationAwareBeanPostProcessor**

如字面意思一样，它定义了两个比较smart的方法：

`Class<?> predictBeanType(Class<?> beanClass, String beanName)`：预测这个processor最终返回的Bean的类型。

`Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)`：预测这个processor使用哪些构造器来创建最终返回的Bean。

除了这两个对得起它名字的smart的方法，该接口还定义了一个非常重要（解决循环依赖）的方法：

`Object getEarlyBeanReference(Object bean, String beanName)`：返回一个还没有完全实例化好的Bean。

**DestructionAwareBeanPostProcessor**

上述的三个接口都是在将Bean的实例化、初始化，而这个接口则是在Bean销毁的时候，进行处理：

```java
//在销毁Bean之前被回调
void postProcessBeforeDestruction(Object bean, String beanName)
```

与DisposableBean的destroy方法和自定义的destory-method一样，该processor也同样只在单例Bean的生命周期中被回调。

**MergedBeanDefinitionPostProcessor**

该接口是指对beanDefinition进行处理，显然这个是Bean生命周期的最早阶段的事情。

```java
//beanDefinition是合并后的最终的beanDefinition
void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
```

**InstantiationAwareBeanPostProcessorAdapter**

这个类其实什么都没有做，它对SmartInstantiationAwareBeanPostProcessor / InstantiationAwareBeanPostProcessor / BeanPostProcessor这三个接口中的8个方法都进行了“什么都不做”的实现（即实现方法都是空的，与接口中方法的默认行为一致），这样子类只需要重写真正关心的方法就可以了，而不用再处理其他不关心的方法。这个类只是Spring内部实现时使用，我们关心哪个方法，直接实现对应的接口就可以了，不用继承该类。

BeanPostProcessor接口及其子接口提供了对Bean生命周期的各个阶段的扩展。这些扩展到底有哪些呢？就在它们的实现类中。

**InitDestroyAnnotationBeanPostProcessor**

从名字猜测，init，destroy，annotation这三个关键词就解释了它的作用：调用Bean中用注解配置的init和destroy方法，可以通过如下方法来指定哪个注解标识的方法是init方法，哪个注解标识的是destroy方法：

```java
void setInitAnnotationType(Class<? extends Annotation> initAnnotationType)
void setDestroyAnnotationType(Class<? extends Annotation> destroyAnnotationType)
```

但通常来讲，init方法通常使用@PostConstruct，destroy方法通常使用@PreDestroy。

实现思路：在postProcessBeforeInitialization中调用Bean的init方法，在postProcessAfterInitialization中调用Bean的destroy方法。

而Bean的init和destroy方法到底是什么，则从postProcessMergedBeanDefinition中获取。

**CommonAnnotationBeanPostProcessor**

这个实现非常重要。

首先它继承自InitDestroyAnnotationBeanPostProcessor，具备了调用Bean中用注解配置的init和destroy方法，它设置的init和destroy的注解分别为@PostConstruct和@PreDestroy。

此外，它实现了InstantiationAwareBeanPostProcessor，说明它会在Bean实例化之前和之后，Bean初始化之后做一些事情。具体的事情则在对应的实现方法中，主要是在postProcessPropertyValues中，通过Bean的定义，拿到有哪些属性标注了@Resource注解（field.isAnnotationPresent\(Resource.class\)），然后注入到Bean中（field.set\(target, getResourceToInject\(target, requestingBeanName\)\);）。具体实现比较复杂，涉及到缓存和线程安全等问题。

```java
public PropertyValues postProcessPropertyValues(
            PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
    InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Injection of resource dependencies failed", ex);
    }
    return pvs;
}
```

即CommonAnnotationBeanPostProcessor可以将我们在类中使用@Resource注解来声明的属性Bean，真正地注入到该类中。

todo 这些processor的注册问题

**AutowiredAnnotationBeanPostProcessor**

这个实现同样非常重要。从它的名字autowired，annotation可以看出，它负责处理表示自动装配的注解，这些注解默认包括：Spring提供的@Autowired和@Value，以及JSR-330中的@Inject。也可以通过void setAutowiredAnnotationType\(Class&lt;? extends Annotation&gt; autowiredAnnotationType\)进行自定义这些注解。

主要是在postProcessPropertyValues中，通过Bean的定义，拿到有哪些属性标注了设置的表示自动装配的注解（field.isAnnotationPresent\(XXXXX.class\)），然后注入到Bean中（field.set\(target, getResourceToInject\(target, requestingBeanName\)\);）。

即AutowiredAnnotationBeanPostProcessor可以将我们在类中使用@Autowired/@Value/@Inject注解来声明的属性Bean，真正地注入到该类中。

**RequiredAnnotationBeanPostProcessor**

负责检测标注了@Required的属性是否真的有值，没有值则抛异常。实现也非常简单，在postProcessPropertyValues方法中，检查PropertyValues，如果一个属性加了@Required注解，则检查它对应的value是否为null。

上述的AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor是通过在xml配置如下信息来开启的：

```xml
<context:annotation-config/>
<context:component-scan base-package="com.maxwell.learning.spring"/>
```

并且，注解方式的注入是先于在xml配置的Bean注入执行的，所以xml的注入配置会覆盖注解的注入配置。

还有这样一个问题：假如配置了多个BeanPostProcessor，它执行的先后顺序是什么呢？

所有的BeanPostProcessor的最终实现类都会实现PriorityOrdered接口，它定义了int getOrder\(\);方法来表示该processor的优先级：order越大，优先级越低；order越小，优先级越高。

上面的几个实现类：CommonAnnotationBeanPostProcessor的order为Integer.MAX\_VALUE-3，AutowiredAnnotationBeanPostProcessor的order为Integer.MAX\_VALUE-2，RequiredAnnotationBeanPostProcessor的order为Integer.MAX\_VALUE-1，所以他们的执行的先后顺序为：CommonAnnotationBeanPostProcessor &gt; AutowiredAnnotationBeanPostProcessor &gt; RequiredAnnotationBeanPostProcessor。

**ApplicationContextAwareProcessor**

负责处理Bean实现的Aware的6个接口，是在Bean已经完全初始化好之后，主要逻辑如下：

```java
@Override
public Object postProcessBeforeInitialization(final Object bean, String beanName){
   if (bean instanceof Aware) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```
**ServletContextAwareProcessor**
负责处理Bean实现的两个Aware接口：ServletContextAware和ServletConfigAware。
**BeanValidationPostProcessor**
负责处理Bean中标注的JSR-303中用于验证Bean属性值的注解，如@NotNull，@Min\(value\)，@Pattern\(value\)等等。
**MethodValidationPostProcessor**
负责处理方法的参数中标注的JSR-303中用于验证的注解，如：
```java
public @NotNull Object myValidMethod(@NotNull String arg1, @Max(10) int arg2)
```
它的实现方式是为方法加上AOP增强，增强逻辑即为验证逻辑。
**AbstractAutoProxyCreator**
这个实现类非常重要，是实现AOP的地方。
如果某个Bean使用AOP进行了功能增强，那么AbstractAutoProxyCreator会为该Bean生成代理，生成代理这个操作需要在Bean实例化之前进行，因此，它的实现主要在postProcessBeforeInstantiation方法中。
AOP相关的功能实现，都在AbstractAutoProxyCreator及其子类当中。
**ScheduledAnnotationBeanPostProcessor**
负责处理Bean中的使用@Scheduled注解标注的方法。一般这样的方法都是具体的业务逻辑，所以需要在Bean完全初始化好之后进行处理，即在postProcessAfterInitialization中处理。
**AsyncAnnotationBeanPostProcessor**
负责处理Bean中的使用@Async注解标注的方法。
这些processor，有的是参与到创建Bean的过程中，有的是对Bean功能的增强，大大减少了BeanFactory的工作。当然，还需要告诉BeanFactory到底有哪些processor，一般在具体的BeanFactory实现类中，会选择性地添加这些proccssor，以实现特定的功能。
## 参考
---
[Spring的BeanFactoryPostProcessor和BeanPostProcessor](https://blog.csdn.net/caihaijiang/article/details/35552859)
[Spring源代码分析（4）---BeanFactoryPostProcessor  ](https://blog.csdn.net/turkeyzhou/article/details/2915438?utm_source=blogxgwz0)
[Spring核心——官配BeanFactoryPostProcessor](https://my.oschina.net/chkui/blog/1854771)
[浅谈Spring的PropertyPlaceholderConfigurer](https://blog.csdn.net/qq_26222859/article/details/51104582?utm_source=blogxgwz0)
[Spring Core Container 源码分析五：@Autowired](https://www.shangyang.me/2017/04/05/spring-core-container-sourcecode-analysis-annotation-autowired/)
[Spring内部的BeanPostProcessor接口总结          ](https://fangjian0423.github.io/2017/06/20/spring-bean-post-processor/)
[JSR 303 - Bean Validation 介绍及最佳实践](https://www.ibm.com/developerworks/cn/java/j-lo-jsr303/index.html)