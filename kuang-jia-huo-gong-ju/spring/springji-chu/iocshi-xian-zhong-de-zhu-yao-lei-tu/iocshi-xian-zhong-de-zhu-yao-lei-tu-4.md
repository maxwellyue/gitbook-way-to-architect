##  **BeanFactoryPostProcessor**

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

即BeanFactoryPostProcessor主要是用来在Bean实例化之前（加载完Bean定义之后），修改Bean的属性的，但是并不仅限于此，一切想在实例化Bean之前想做的事情，都可以在这里做。

这里没有画它的实现类，因为它的实现类通常主要功能与BeanFactoryPostProcessor无关，只是需要在上面说的这个时机来做一些事情。

比如，PropertyPlaceholderConfigurer实现了该接口，它做的事情是加载程序中的配置文件，然后将Bean定义中出现的占位符替换为真正的配置文件中写的值。

## **BeanPostProcessor**

---

BeanPostProcessor，字面意思就是“Bean之后的处理器”（post可以理解为after），接口如下（注释较长，没有粘过来）：

```java
/**
 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
 * or a custom init-method). The bean will already be populated with property values.
 * The returned bean instance may be a wrapper around the original.
 * 
 * The default implementation returns the given {@code bean} as-is.
 * 
 * @param bean the new bean instance
 * @param beanName the name of the bean
 * @return the bean instance to use, either the original or a wrapped one;
 */
default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
/**
 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
 * or a custom init-method). The bean will already be populated with property values.
 * The returned bean instance may be a wrapper around the original.
 * 
 * In case of a FactoryBean, this callback will be invoked for both the FactoryBean
 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
 * post-processor can decide whether to apply to either the FactoryBean or created
 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
 * This callback will also be invoked after a short-circuiting triggered by a
 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
 * in contrast to all other BeanPostProcessor callbacks.
 * 
 * The default implementation returns the given {@code bean} as-is.
 * 
 * @param bean the new bean instance
 * @param beanName the name of the bean
 * @return the bean instance to use, either the original or a wrapped one;
 */
@Nullable
default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
```

postProcessBeforeInitialization：该方法会在与beanName对应的Bean完成实例化（且已经被填充了属性值）之后，但在其他实例化回调（如该Bean实现了InitializingBean的afterPropertiesSet或者该Bean定制了init-method）之前，进行调用。返回值bean可以是原始Bean（即入参Object bean）的包装类。

postProcessAfterInitialization：该方法会在与beanName对应的Bean完成实例化（且已经被填充了属性值）之后，且在其他实例化回调（如该Bean实现了InitializingBean的afterPropertiesSet或者该Bean定制了init-method）之后，进行调用。返回值bean可以是原始Bean（即入参Object bean）的包装类。

它的实现类同样只是在这样的时机来对Bean做一些处理，但是有一些与容器实现关系密切的实现类。

todo Spring内部的应用









## 参考

---

[Spring的BeanFactoryPostProcessor和BeanPostProcessor](https://blog.csdn.net/caihaijiang/article/details/35552859)

[Spring源代码分析（4）---BeanFactoryPostProcessor  
](https://blog.csdn.net/turkeyzhou/article/details/2915438?utm_source=blogxgwz0)

