## Spring的**ClassPathBeanDefinitionScanner**

---

在Spring中，负责读取Class文件中定义的BeanDefinition的是ClassPathBeanDefinitionScanner。它会扫描类上标记的注解，如果发现符合条件的，就读取它来获取BeanDefinition。这个条件，就是类上标注了特定的注解，这些特定的注解是哪些注解呢？这是通过它的includeFilters变量来记录的，默认情况下，它会将一些特定的注解类型注册到这个变量中：

```java
protected void registerDefaultFilters() {

    //@Component注解，以及“继承”了该注解的@Repository、@Service、@Controller
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
    ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
    try {
        //JSR-250中的@ManagedBean注解，前提是项目引入了javax.annotation这个包
        this.includeFilters.add(new AnnotationTypeFilter(
                ((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
        logger.debug("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
    }
    try {
        //JSR-330中的@Named注解，前提是项目引入了javax.inject这个包
        this.includeFilters.add(new AnnotationTypeFilter(
                ((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
        logger.debug("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - simply skip.
    }
}
```

即它默认从标注了@Component注解（包括“继承”了该注解的@Repository、@Service、@Controller）以及@ManagedBean、@Named的类来读取BeanDefinition。

如果想要改变它的策略，则可以通过自定义includeFilters或者excludeFilters，来定义读取哪些注解，排除哪些注解。

这些类是从Reource中获取到的，有两种方式传递给ClassPathBeanDefinitionScanner。一是在构造器中传入该参数，另一个是调用它的scan\(String...basePackages\)方法，它会将basePackages解析为Reource，逻辑如下：

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
    Set<BeanDefinition> candidates = new LinkedHashSet<>();

    String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
    Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
    for (Resource resource : resources) {
        if (resource.isReadable()) {
            MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
            //class上是否标注了特定的注解
            if (isCandidateComponent(metadataReader)) {
                //BeanDefinition的类型为ScannedGenericBeanDefinition
                ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                sbd.setResource(resource);
                sbd.setSource(resource);
                //[类是可以被单独构造的具体类（即非接口，非抽象类）]或者[类是抽象类，但是含有标注了Lookup的方法]
                if (isCandidateComponent(sbd)) {
                    candidates.add(sbd);
                }    
            }
        }
    }
    return candidates;
}
```

在拿到BeanDefinition后，再去设置每个BeanDefinition的其他信息：

```java
for (BeanDefinition candidate : candidates) {
    //获取作用域以及代理模式
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
    candidate.setScope(scopeMetadata.getScopeName());
    String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
    if (candidate instanceof AbstractBeanDefinition) {
        postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
    }
    if (candidate instanceof AnnotatedBeanDefinition) {
        AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
    }
    if (checkCandidate(beanName, candidate)) {
        BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
        definitionHolder =
                AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
        beanDefinitions.add(definitionHolder);
        registerBeanDefinition(definitionHolder, this.registry);
    }
}
```

（1）作用域以及可能的AOP代理模式。ScopeMetadata有两个信息：一是作用域的名字\(如singleton/prototype等\)，二是ScopedProxyMode，它有4种取值：

* DEFAULT：默认值，等同于NO
* NO：不创建代理
* INTERFACES：使用JDK动态代理
* TARGET\_CLASS：使用CGLIB代理

（2）使用了AnnotationBeanNameGenerator作为Bean名字的生成器，生成逻辑为：①如果注解中使用通过value指定了名字，则使用这个指定的名字，比如@Service\(value="xxxxService"\)，则最终Bean名字为xxxxService；②如果没有指定，则使用短类名，并将第一个字母进行小写作为Bean名字，比如“com.maxwell.example.UserServiceImpl”，则最终Bean名字为userServiceImpl。

其中，postProcessBeanDefinition是设置BeanDefinition的一些默认属性值：

```java
public void applyDefaults(BeanDefinitionDefaults defaults) {
    setLazyInit(defaults.isLazyInit());//默认为false，即懒加载
    setAutowireMode(defaults.getAutowireMode());//默认为不自动装配
    setDependencyCheck(defaults.getDependencyCheck());//默认不检查依赖
    setInitMethodName(defaults.getInitMethodName());
    setEnforceInitMethod(false);
    setDestroyMethodName(defaults.getDestroyMethodName());
    setEnforceDestroyMethod(false);
}
```

（3）processCommonDefinitionAnnotations是进一步对类的其他注解进行解析，设置BeanDefinition的其他属性：①是否标记了@Lazy，设置是否懒加载；②是否标记了@Primary，设置是否为优先使用；③是否有@DependsOn，如果有，则设置BeanDefinition的dependsOn属性；④是否标记了@Role注解，设置BeanDefinition的role属性；⑤是否标记了@Description，设置BeanDefinition的description属性。

参考：[Spring类注册笔记](https://fangjian0423.github.io/2017/06/15/spring-bean-register-note/)

