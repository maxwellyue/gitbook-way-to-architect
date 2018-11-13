## **BeanFactoryPostProcessor**

---

**概念**

BeanFactoryPostProcessor，字面意思就是“Bean工厂之后的处理器”（post可以理解为after），接口如下：

```java
public interface BeanFactoryPostProcessor {
    /**
     * Modify the application context's internal bean factory after its standard
     * initialization. All bean definitions will have been loaded, but no beans
     * will have been instantiated yet. This allows for overriding or adding
     * properties even to eager-initializing beans.
     * @param beanFactory the bean factory used by the application context
     * @throws org.springframework.beans.BeansException in case of errors
     */
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

上面的英文的大意是：在bean factory完成标准的初始化之后，对它进行修改。具体时机为：所有的bean definitions都已经加载完成，但是bean还没有进行实例化。通过该方法，可以对Bean的属性进行修改或者添加（eager是与lazy的反义 ）。

即BeanFactoryPostProcessor主要是用来在Bean实例化之前（加载完Bean定义之后），修改Bean的属性的，但是并不仅限于此，一切想在实例化Bean之前想做的事情，都可以在这里做。![](/assets/屏幕快照 2018-10-29 下午11.04.10.png)

**PropertyResourceConfigurer**

负责处理配置文件中的信息，即将配置文件中配置的值解析出来传递给Bean Definition。一般配置Bean的时候，有两种方式，不同的配置方式的具体处理由两个实现类来具体负责。

一是在配置Bean的时候使用占位符，而真正的值写在xxx.properties配置文件中，如：

```xml
//使用占位符进行配置
<bean id="user" class="com.maxwell.learning.spring.User">
    <property name="name" value="${user.name}"/>
    <property name="age" value="${user.age}"/>
</bean>
```

配置文件config.properties：

```
user.name='maxwell'
user.age=20
```

这种形式（`${...}`）的配置方式，是由PropertyPlaceholderConfigurer来处理的。

二是在配置Bean的时候，设置了默认值，但在配置文件中又重写赋值，如：

```java
<bean id="user" class="com.maxwell.learning.spring.User">
    <property name="name" value="maxwell"/>
    <property name="age" value="20"/>
</bean>
```

配置文件config.properties：

```
user.name='max'
user.age=30
```

这种形式（`beanName.property=value`）的配置方式，是由PropertyOverrideConfigurer来处理的。

此外，PropertyPlaceholderConfigurer除了会用\*.properties文件中的参数去替换占位符的内容，还会使用环境变量（System.getProperty\(key\)）中的参数去替换。如果一个参数在配置文件中和系统环境变量中都存在，那么默认会使用\*.properties中的参数来替换配置中的占位符，可以使用PropertyPlaceholderConfigurer::systemPropertiesMode来修改这个行为。

**PropertySourcesPlaceholderConfigurer**

是PropertyPlaceholderConfigurer的升级版，可以额外处理环境Environment和@Value注解中配置信息。也就是它可以根据不同环境来设置来为Bean去加载不同的配置信息。

这些PropertyXxxxConfigurer是BeanFactoryPostProcessor的一类实现或者一类应用，就是去将正确的配置信息设置到Bean Definition中。

**CustomScopeConfigurer**

从名字来看，它是用来自定义作用域的。Spring默认的作用域有singleton、prototype、request、session、global session。假如这些作用域不符合应用场景，比如想要一个作用域为Thread时（即想要每个线程有一个Bean实例），则可以通过CustomScopeConfigurer来自定义作用域。

上面说到的PropertyXxxxConfigurer负责在BeanFactory创建好之后，在postProcessBeanFactory\(ConfigurableListableBeanFactory beanFactory\)方法，对将要加载或实例化的Bean的Definition传入必要的配置信息。那么自定义作用域为什么也要在这个时机来做呢？

这是因为，Bean的作用域决定了在BeanFactory在getBean的时候到底用什么策略去拿到Bean，是创建实例，还是用原来缓存的等等。比如，我们假如自定义Thread作用域，那么BeanFactory在getBean的时候就需要看当前Thread是不是已经有Bean了，有就直接拿，没有就为当前Thread创建Bean。这个逻辑，需要告知BeanFactory，以便它进行之后的逻辑。

**CustomEditorConfigurer**

从名字来看，看不出来是edit什么，如果叫CustomPropertyEditorConfigurer就更合适了，其实是对属性进行编辑，即属性编辑配置器（由字符串转换为任意Java类型）。

Spring了已经内置了很多属性编辑器，但是如果不能满足需求，就要使用它来完成。

**DeprecatedBeanWarner**

对标注了@Deprecated的Bean打出warning日志。

**BeanDefinitionRegistryPostProcessor**

该接口额外定义了一个方法：所有的常规的BeanDefinition都已经被加载，但是还没有Bean被实例化，所以可以在这里添加更多的BeanDefinition。

```java
void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
```

**ConfigurationClassPostProcessor**

它是BeanDefinitionRegistryPostProcessor的实现类，从它的名字叫ConfigurationClass处理器，ConfigurationClass就是标注了@Configuration的class。也就是，它负责从标注了@Configuration的class中解析出BeanDefinition，然后传给BeanDefinitionRegistry。其实，不仅仅是标注了@Configuration的class，它还会解析@Component，@ComponentScan，@Import等注解。

> @Service，@Controller，@Repository其实都是@Component
关键逻辑代码：

```java
//
if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
	configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
}
//
if (isFullConfigurationCandidate(metadata)) {
	beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);
}
else if (isLiteConfigurationCandidate(metadata)) {
	beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);
}
//
public static boolean isFullConfigurationCandidate(AnnotationMetadata metadata) {
	return metadata.isAnnotated(Configuration.class.getName());
}
public static boolean isLiteConfigurationCandidate(AnnotationMetadata metadata) {
	for (String indicator : candidateIndicators) {
		if (metadata.isAnnotated(indicator)) {
			return true;
		}
	}
	... ... 	
	return metadata.hasAnnotatedMethods(Bean.class.getName());
	... ...
}
//
static {
	candidateIndicators.add(Component.class.getName());
	candidateIndicators.add(ComponentScan.class.getName());
	candidateIndicators.add(Import.class.getName());
	candidateIndicators.add(ImportResource.class.getName());
}
```

可以看出，这个类是非常重要的，它解析了我们在class中通过注解定义的Bean。

## 参考

---

[Spring的BeanFactoryPostProcessor和BeanPostProcessor](https://blog.csdn.net/caihaijiang/article/details/35552859)

[Spring源代码分析（4）---BeanFactoryPostProcessor  ](https://blog.csdn.net/turkeyzhou/article/details/2915438?utm_source=blogxgwz0)

[Spring核心——官配BeanFactoryPostProcessor](https://my.oschina.net/chkui/blog/1854771)

[浅谈Spring的PropertyPlaceholderConfigurer](https://blog.csdn.net/qq_26222859/article/details/51104582?utm_source=blogxgwz0)

[Spring Core Container 源码分析五：@Autowired](https://www.shangyang.me/2017/04/05/spring-core-container-sourcecode-analysis-annotation-autowired/)

[Spring内部的BeanPostProcessor接口总结          ](https://fangjian0423.github.io/2017/06/20/spring-bean-post-processor/)

[JSR 303 - Bean Validation 介绍及最佳实践                        ](https://www.ibm.com/developerworks/cn/java/j-lo-jsr303/index.html)

[spring boot 加载过程分析--ConfigurationClassPostProcessor  
](https://juejin.im/post/5b51861e5188251b166ef991)
