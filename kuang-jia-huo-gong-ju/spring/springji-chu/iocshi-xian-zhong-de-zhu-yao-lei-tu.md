# BeanFactory

---

BeanFactory的定义：

```java
public interface BeanFactory {
    //FactoryBean的转义符
    String FACTORY_BEAN_PREFIX = "&";
    
    //获取Bean的方法
    Object getBean(String name) throws BeansException;
    <T> T getBean(String name, Class<T> requiredType);
    <T> T getBean(Class<T> requiredType) throws BeansException;
    Object getBean(String name, Object... args) throws BeansException;
    
    //工具方法
    boolean containsBean(String name);
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;    
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;    
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    String[] getAliases(String name);
}
```

BeanFactory的直接继承接口（二级接口），有`HierarchicalBeanFactory`、`AutowireCapableBeanFactory`、`ListableBeanFactory`。

* `HierarchicalBeanFactory`

* * 为实现bean工厂的层级关系提供支持，定义了`BeanFactory getParentBeanFactory()`方法；比如常见的Web应用中，ContextLoaderListener创建的容器为父容器，而DispatcherServlet创建的容器为子容器。
  * 容器有父子之分，父容器对子容器可见，子容器对父容器不可见。

* `AutowireCapableBeanFactory`

* * 提供自动装配Bean的能力，主要方法为`void autowireBean(Object existingBean)`。
  * ```xml
    //引入Autowire之前
    <bean id="example" class="com.maxwell.learning.spring.Example">
        <property name="a" ref="nameA"/>
        <property name="b" ref="nameB"/>
        <property name="c" ref="nameC"/>
    </bean>
    //引入Autowire之后
    <bean id="example" class="com.maxwell.learning.spring.Example" autowire="byName">
    ```
  * autowire包含4中模式：

  * * no：默认值，Spring建议使用该值，即我们在配置中显式指明其依赖，而不是靠Spring根据类型或名字来判断
    * byName：根据依赖Bean的名字去获取依赖Bean
    * byType：根据依赖Bean的类型去获取依赖Bean
    * constructor：在构造时，按照依赖Bean的类型去获取依赖Bean

  * 
  * 这时候可能需要我们为这个Bean写很多类似&lt;value&gt;
  * * 12

* `ListableBeanFactory`

* * 1
  * 1
  * 1

12sdfd



# 

## 参考

---

[Spring BeanFactory 类图详解](https://blog.csdn.net/qq_34090008/article/details/78772189)

[Spring 父子容器](http://wangxinchun.iteye.com/blog/2341197)

[Spring AutowireCapableBeanFactory学习](https://www.jianshu.com/p/d564335fafab)

[Spring源码学习--@Autowired注解和启动自动扫描的三种方式  
](https://blog.csdn.net/u013412772/article/details/73741710)

