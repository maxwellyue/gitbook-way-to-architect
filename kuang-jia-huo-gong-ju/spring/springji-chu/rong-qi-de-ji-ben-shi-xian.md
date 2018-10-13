# Ioc容器的实现原理

---

Ioc，即Inversion of Control，控制反转，在没有引入IOC容器之前，假如对象A依赖于对象B，那么对象A在初始化或者运行到某一点的时候，自己必须主动去创建对象B或者使用已经创建的对象B。无论是创建还是使用对象B，控制权都在自己手上。在引入IOC容器之后，对象A与对象B之间失去了直接联系，所以，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方。

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









## 参考

---

[1000行代码读懂Spring（一）- 实现一个基本的IoC容器](https://my.oschina.net/flashsword/blog/192551)

[深入理解Spring系列之二：BeanDefinition解析](https://www.jianshu.com/p/8d92147653c0)





  


  








