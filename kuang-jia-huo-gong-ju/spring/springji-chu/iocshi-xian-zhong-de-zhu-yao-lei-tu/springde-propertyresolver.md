## PropertyResolver

---

![](/assets/屏幕快照 2018-11-08 下午10.24.19.png)

**PropertyResolver**

这里的property是配置的意思，而非之前PropertyAccessor或者PropertyEditor中的属性的意思。

所以PropertyResolver是用来读取或者解析程序的配置信息的。需要注意的是，PropertyResolver并不仅仅只是通过String类型的key来获取String类型的value，它获取到的value可以是任意Java类型。

```java
public interface PropertyResolver {

    boolean containsProperty(String key);

    String getProperty(String key);

    String getProperty(String key, String defaultValue);

    <T> T getProperty(String key, Class<T> targetType);

    <T> T getProperty(String key, Class<T> targetType, T defaultValue);

    String getRequiredProperty(String key) throws IllegalStateException;

    <T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;

    String resolvePlaceholders(String text);

    String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;
}
```

**ConfigurablePropertyResolver**

 可配置的配置解析器，内部持有一个ConversionService，以便在需要的时候进行类型转换。要注意，这里的Configurable并不是可以去设置配置项，而是去配置在获取配置时的一些信息，主要包括：占位符、分隔符的格式，是否有必须的配置项等。

```java
public interface ConfigurablePropertyResolver extends PropertyResolver {

    //内部持有类型转换服务
    ConfigurableConversionService getConversionService();
    void setConversionService(ConfigurableConversionService conversionService);

    //设置占位符的前缀
    void setPlaceholderPrefix(String placeholderPrefix);

    //设置占位符的后缀
    void setPlaceholderSuffix(String placeholderSuffix);

    //设置分隔符，比如“a,b,c”，如果想要将该字符串转换为数组或集合，可以设置分隔符为逗号(,)
    void setValueSeparator(@Nullable String valueSeparator);

    //设置在遇到不能解析的嵌套的属性时，是否抛出异常
    void setIgnoreUnresolvableNestedPlaceholders(boolean ignoreUnresolvableNestedPlaceholders);

    //设置必须的属性有哪些，当这些属性不存在时，将抛出异常
    //由下面的validateRequiredProperties方法进行校验
    void setRequiredProperties(String... requiredProperties);

    //校验通过setRequiredProperties设置的属性是否可以解析到，并且不能为空，否则将抛异常
    void validateRequiredProperties() throws MissingRequiredPropertiesException;

}
```

**AbstractPropertyResolver**

ConfigurablePropertyResolver的抽象的实现类，实现了大部分方法。

**PropertySourcesPropertyResolver**

ConfigurablePropertyResolver的最终实现类，其内部持有一个PropertySources实例，从中获取配置。

**Environment**

表示程序当前运行的环境（比如生产/预发/测试/开发等）。 

```java
public interface Environment extends PropertyResolver {

    String[] getActiveProfiles();

    String[] getDefaultProfiles();

    boolean acceptsProfiles(String... profiles);
}
```

**ConfigurableEnvironment**

可配置的环境，相比Environment，提供set和add方法。

**AbstractEnvironment**

Environment的抽象实现类。

**StandardEnvironment、StandardServletEnvironment**

StandardEnvironment在非WEB应用中使用，StandardServletEnvironment则在WEB应用中使用。

**EnvironmentCapable**

它与上面介绍的其实不在一个范畴，放在这里是因为与环境相关。它表示一个组件可以关联一个Environment。

```java
public interface EnvironmentCapable {

    //Return the Environment associated with this component.
    Environment getEnvironment();
}
```

1111



