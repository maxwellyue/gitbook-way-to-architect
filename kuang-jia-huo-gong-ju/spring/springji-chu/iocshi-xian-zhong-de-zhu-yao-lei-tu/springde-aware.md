# Aware

---

容器管理的Bean一般不需要了解容器的状态和直接使用容器，但是在某些情况下，是需要在Bean中直接对IOC容器进行操作的，这时候，就需要在Bean中设定对容器的感知。Spring IOC容器也提供了该功能，它是通过特定的Aware接口来完成的。

```java
public interface Aware {
}
```

Aware是一个标识接口，实现该接口，就意味着用户可以从容器中获取一些资源，并在特定的时机添加一些回调。子类或子接口的方法相当于一个回调函数，IoC容器在执行到对应的方法后，会调用其中的逻辑。

**BeanNameAware**

获取Bean的beanName（明明方法是setBeanName，为什么是获取呢？这个获取的意思，是相对用户来说，IoC去执行setBeanName这个方法，而在这个方法中，我们就相当于获取到了这个name）

```java
//接口
public interface BeanNameAware extends Aware {
    void setBeanName(String name);
}
//举例子
public class UserService implements BeanNameAware{
   @Override
   void setBeanName(String name){
      System.out.print("my bean name is :" + name);
   }
}
//IoC源码中的回调
 if (bean instanceof Aware) {
    if (bean instanceof BeanNameAware) {
       ((BeanNameAware) bean).setBeanName(beanName);
```
**BeanFactoryAware**
获取当前Bean Factory：`void setBeanFactory(BeanFactory beanFactory)`
**ApplicationContextAware**
获取当前的Application Context：`void setApplicationContext(ApplicationContext applicationContext)`
**MessageSourceAware**
获取MessageSource：`void setMessageSource(MessageSource messageSource);`
**ApplicationEventPublisherAware**
获取应用事件发布器：`void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher);`
**ResourceLoaderAware**
获取资源加载器：`void setResourceLoader(ResourceLoader resourceLoader);`
## 参考
---
[Spring实现Aware接口，完成对IOC容器的感知](https://blog.csdn.net/ilovejava_2010/article/details/7953582)
[Spring Aware 相关接口的使用](https://hacpai.com/article/1514947996486)