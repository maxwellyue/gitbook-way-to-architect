# Ioc容器的实现原理

---

Ioc，即Inversion of Control，控制反转，在没有引入IOC容器之前，假如对象A依赖于对象B，那么对象A在初始化或者运行到某一点的时候，自己必须主动去创建对象B或者使用已经创建的对象B。无论是创建还是使用对象B，控制权都在自己手上。在引入IOC容器之后，对象A与对象B之间失去了直接联系，所以，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方。

## 基本概念

---

下面，我们就从最简单的概念入手，一步一步看Spring是如何实现Ioc容器的。

**Map数据结构**

要达到这样的效果，那么对象A和B都应该交给Ioc容器来管理，所以，正如Ioc容器这个名字所指示的一样，它是一个容器，用来盛放对象，这些对象，我们称之为Bean。在Java中，容器分为集合和Map两大类，在Ioc的场景下，我们要将对象放入容器，然后再通过一个标识符（比如全类名）去拿到这个对象，很显然，使用Map&lt;String, Object&gt;是较为合理的。

到这一步，我们直到了Ioc容器中最重要的概念：容器和Bean，它们之间的关系为：容器使用Map这种数据结构来存取Bean。

**BeanFactory和FactoryBean**

确定了将对象存放在Map中，现在就可以考虑将对象放入其中了。由于程序中存在大量的对象（即Bean）需要Ioc容器来管理，因此我们需要一个生产Bean的工厂，让它来负责对象的创建，在Spring中这个工厂叫做BeanFactory，它定义了Ioc容器的一个最基本的功能：获取Bean，即BeanFactory至少要有一个如下方法：

```java
<T> T getBean(Class<T> requiredType)
```

对于要放入容器的Bean而言，可能会有这样的需求：不让BeanFactory来统一生产，而是用户自定义生产逻辑，让用户灵活配置对象的具体创建逻辑，这就是FactoryBean的使命，它定义了`T getObject()`方法，如果某个类实现了这个接口，那么通过BeanFactory的getBean\(\)获取到的对象是getObject\(\)这个方法所返回的对象。

到这一步，我们知道，BeanFactory可以用来获取对象，且FactoryBean可以自定义对象的创建逻辑。

**BeanDefinition**

BeanFactory要想获取对象，或者说要想从Map中获取对象，必须通过一个标识符或key来完成。这个标识符通常是类的全限定名。但是假如在程序中需要某个类的两个实例，这时候仅仅通过类的全限定名就无法区分，这时候就需要给这两个实例分别起一个名字。

BeanFactory要想获取对象，首先需要将对象创建出来，然后才能获取。在应用中，有些类我们希望是单例的，即每次BeanFactory.getBean\(\)都是获取的同一个实例，而有时又希望每次BeanFactory.getBean\(\)时是new的新的实例。

又或者，我们不想在应用程序启动时就将某些对象创建，而是希望在真正用到的时候，再去创建。

我们需要将这些信息，如名字，是否单例，是否懒加载等等，与程序中某个类绑定在一起，作为这个类的附加属性，告知给BeanFactory这个工厂，然后它才能按照我们的意图来生产Bean。

这就是Spring中的BeanDefinition，它与某个具体的类对应，里面包含了类名、构造函数参数列表、依赖的bean、是否是单例类、是否是懒加载等，其实就是将Bean的定义信息存储到这个BeanDefinition中，之后对Bean的操作就直接对BeanDefinition进行，例如拿到这个BeanDefinition后，可以根据里面的类名、构造函数，使用反射进行对象创建。

到这一步，我们知道，BeanDefinition就是我们应用中的某个类的包装，记录了类名，是否单例，是否懒加载，依赖哪些类等信息。

**Profile**

一般而言，我们的应用程序都是区分环境的，比如生产/预发/测试/开发等。假如我们想要某些对象只在某种特定的环境下进行加载，或者在不同的环境中我们希望同一个类的实例在某些属性上可以配置不同的值，这就是Spring中的Profile，用来区分在不同的情况下的BeanDefinition的配置，相当于给BeanDefinition进行了分组，在不同的情况下让Spring去加载不同分组的BeanDefinition。

到这一步，我们知道，Profile就是我们对BeanDefinition进行分组，以便在不同情况下（通常是不同环境）下加载不同的Bean。

**Resource和EncodedResource**

BeanFactory负责创建对象或者从中获取对象。或者，从最根本的说起，Ioc容器负责管理对象。但它要管理哪些对象，BeanFactory又要创建哪些对象？我们必须通过一定的方式来告诉它，最常见的是写在XML文件中，然后让BeanFactory通过某种方式去将我们定义在XML中的Bean创建出来。但是假如这些Bean是定义在某个远程服务器上，需要我们通过url去读取，再或者假如这些Bean就是一些jar文件（虽然只有类，但是如名字、是否懒加载等信息可以采用默认值的方式形成BeanDefinition），所以，需要一个对底层不同来源的资源的封装，这就是Spring中的Resource。对于不同来源的资源，Resource有不同的实现，如文件（FileSystemResource）、ClassPath资源（ClassPathResource）、URL资源（UrlResource）、InputStream资源（InputStreamResource），Byte数组（ByteArrayResource）等。

由于这些不同来源的资源可能存在编码问题，因此需要一个处理编码的工具，这就是Spring中的EncodedResource。

到这一步，我们知道，Resource是对所有资源文件进行统一处理，不仅是Bean的定义这些资源，其他的资源如配置文件等也可以通过Resource来处理。而EncodedResource则是来处理这些资源的编码问题的工具类。

**BeanDefinitionReader**

现在，有了Bean的定义所在的资源Resource，那么，如何将这些不同来源的Resource转换为BeanDefinition呢？

即我们需要一个从Resource中获取BeanDefinition的方法：

```java
List<BeanDefinition> loadBeanDefinitions(Resource resource)
```

获取到Bean之后，还需要把它交给Ioc容器，因此还需要一个传递给Ioc容器的方法：

```java
void registerBeanDefinitions(List<BeanDefinition> beanDefinitions)
```

这就是Spring中的BeanDefinitionReader，它负责从Resource获取BeanDefinition，并将BeanDefinition放在Map&lt;String, Object&gt;这个数据结构中。

**ObjectFactory**

前面说到，当我们调用BeanFactory.getBean\("bean name"\)的时候，就可以得到一个对象的实例化对象。但是getBean\(\)需要做的事情实在太多了：如果是已经创建的单例Bean，那么getBean\(\)只需要从Map中取出来就可以了；如果是未创建的单例Bean，则需要去真正的new实例；如果这个Bean还有很多依赖，又要去加载它的依赖Bean。getBean\(\)需要一个助手来完成“真正地new实例”这个操作，如果是单例，只需要执行一次就足够，如果是prototype，则每次都要执行，而ObjectFactory就是这个助手，它是某个普通对象的工厂，负责生产一个具体的实例，它仅仅包含一个方法：`T getObject()` ，这里的getObject中的“get”通常的语义为“create”，即创建一个类的实例。

**不止一个Map**

那么，上面提到的这个Map到底在哪呢？上面说到，BeanFactory负责生产Bean，当应用程序需要某个Bean的时候，或者某个Bean需要另外一个Bean的时候，都去找BeanFactory去要，因此，这个Map让BeanFactory来持有，是非常合理的。在Spring中，也正是BeanFactory实际持有这个Map。

现在来看这样一种情况：假如我们程序中写了两个类A，B，且A依赖B，B依赖A，代码看起来是这样的，且Spring已经获取到了A和B对应的ABeanDefinition和BBeanDefinition，知道了A和B是循环依赖的关系，且A和B都只需要单例即可。

```java
public class A {
    private B b; //由Ioc容器注入B的实例
}

public class B {
    private A a; //由Ioc容器注入A的实例
}
```

这个时候，假设第一次执行BeanFactory.getBean\("A"\)，发现需要注入B，BeanFactory又去getBean\("B"\)，但是此时发现，B又需要注入A。这好像是一个无法停止的过程。

要解决这个问题，其实有这样一个思路：先执行a = new A\(\)、b = new B\(\)，拿到两个实例，再将实例a和b分别赋给各自的属性。要实现这样的效果，还需要一个Map来存放最初刚刚new出来但是还没set属性的实例。仅仅这样还不够，我们还需要一个Map来存放用来new实例的ObjectFactory。

最终我们需要3个Map来解决循环依赖的问题：（实际Spring中singletonFactories为ConcurrentHashMap）

```java
//Map的key为bean name

//用来new实例的ObjectFactory
Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>();
//new出来还没有set属性的Bean
Map<String, Object> earlySingletonObjects = new HashMap<>();
//已经set属性的Bean
Map<String, Object> singletonObjects = new HashMap<>();
```

假设第一次执行BeanFactory.getBean\("A"\)，则整个流程如下：![](/assets/屏幕快照 2018-10-14 下午9.37.07.png)到这一步，我们知道，Spring中的BeanFactory为了解决循环依赖的问题，在创建阶段，借助了另外两个Map来存储不同时期的Bean。

TODO 



## 参考

---

[1000行代码读懂Spring（一）- 实现一个基本的IoC容器](https://my.oschina.net/flashsword/blog/192551)

[BeanFactory和FactoryBean区别](https://my.oschina.net/wenbo123/blog/1590892)

[Spring IOC 容器源码分析](https://javadoop.com/post/spring-ioc)

[深入理解Spring系列之二：BeanDefinition解析](https://www.jianshu.com/p/8d92147653c0)

[详解Spring中的Profile](https://www.jianshu.com/p/948c303b2253)

[Spring IOC 容器源码分析 - 循环依赖的解决办法](https://segmentfault.com/a/1190000015221968)

