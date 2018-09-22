# JSR-330标准注解

Java依赖注入标准（JSR-330，Dependency Injection for Java）1.0 规范主要是面向依赖注入使用者，而对注入器实现、配置并未作详细要求。目前 Spring 、Guice 已经开始兼容该规范，JSR-299（Contexts and Dependency Injection for Java EE platform，参考实现 Weld ）在依赖注入上也使用该规范。JSR-330 规范并未按 JSR 惯例发布规范文档，只发布了规范 API 源码。

从Spring 3.0开始，Spring开始支持JSR-330标准的注解。这些注解和Spring注解扫描的方式是一直的，开发者只需要引入`javax.inject`即可。

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>xxx</version>
</dependency>
```

JSR-330中的标准注解与Spring中的注解的对应关系如下：

| Spring | JSR-330 | 说明 |
| --- | --- | --- |
| @Autowired | @Inject | @Inject注解没有required属性，但是可以通过Java 8的Optional取代 |
| @Component | @Named | JSR\_330标准并没有提供复合的模型，只有一种方式来识别组件 |
| @Scope\(“singleton”\) | @Singleton | JSR-330默认的作用域类似Spring的prototype，然而，为何和Spring的默认保持一致，JSR-330标准中的Bean在Spring中默认也是单例的。如果要使用非单例的作用域，开发者应该使用Spring的@Scope注解。java.inject也提供一个@Scope注解，然而，这个注解仅仅可以用来创建自定义的作用域时才能使用。 |
| @Qualifier | @Qualifier/@Named | javax.inject.Qualifier仅仅是一个元注解，用来构建自定义限定符的。而String的限定符（比如Spring中的@Qualifier）可以通过javax.inject.Named来实现 |
| @Value | - | 不等价 |
| @Required | - | 不等价 |
| @Lazy | - | 不等价 |
| ObjectFactory | Provider | javax.inject.Provider是SpringObjectFactory的另一个选择，通过get\(\)方法来代理，Provider可以和Spring的@Autowired组合使用 |

**为什么要使用JSR-330提供的标准注解**

JSR-330相当于接口，而Spring是一种实现，在编程中一般面向接口，而不依赖具体实现。

## 参考

[Spring核心技术（十）——JSR-330标准注解    
](https://blog.csdn.net/EthanWhite/article/details/51879871)[Java 依赖注入标准（JSR-330）简介](https://blog.csdn.net/DL88250/article/details/4838803)

