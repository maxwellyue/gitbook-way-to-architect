第六步：registerBeanPostProcessors\(beanFactory\)：注册BeanPostProcessor。

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

第五步中，说PostProcessorRegistrationDelegate主要有两个功能，第一个功能是回调所有的BeanFactoryPostProcessor，第二个功能就是这一步中的注册BeanPostProcessor。

代码过长，注释详见：

主要逻辑如下：









