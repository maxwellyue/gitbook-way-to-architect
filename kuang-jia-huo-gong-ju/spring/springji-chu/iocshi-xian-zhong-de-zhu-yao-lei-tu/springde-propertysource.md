# Spring的PropertySource

---

![](/assets/屏幕快照 2018-11-10 下午12.19.04.png)

**PropertySource**

PropertySource意为属性源，它只有两个变量，且均为final类型。既然是属性源，那么就一定可以从中获取某个属性的值，即含有getProperty方法，它的主要方法如下：

```java
public abstract class PropertySource<T> {

    //表示这个属性源的名字
    protected final String name;
    //表示这个属性源的源
    protected final T source;

    public PropertySource(String name, T source) {
        this.name = name;
        this.source = source;
    }

    public PropertySource(String name) {
        this(name, (T) new Object());
    }

    public String getName() {
        return this.name;
    }

    public T getSource() {
        return this.source;
    }

    public boolean containsProperty(String name) {
        return (getProperty(name) != null);
    }

    @Nullable
    public abstract Object getProperty(String name);
}
```

 **EnumerablePropertySource**

 Enumerable意为可列举的，可枚举的，所以，他在PropertySource的基础上增加了获取所有属性名的方法：

```java
//返回source中所有属性的名字
public abstract String[] getPropertyNames();
```

**MapPropertySource**

从名字中可以看出，它是采用Map来表述source，即source即为Map&lt;属性，属性值&gt;。

**PropertiesPropertySource**

从名字可以看出，它是采用java.util.Properties来表述source，即source为 Properties，而Properties继承自HashTable，实质也是一个Map。与MapPropertySource区别在于，构造时传入一个Properties即可。

**SystemEnvironmentPropertySource**

构造时传入系统环境变量 ，专门处理从系统环境变量中获取属性值。

 其他实现类：TODO

总结：假如“一个地方”存放着很多&lt;属性名，属性值&gt;这样的键值对，那“这个地方”就是上面说的source，这个source可以是map，也可以是Properties等等。而PropertySource则表示&lt;某个地方，这个地方所有的&lt;属性名， 属性值&gt;键值对&gt;。

**PropertySources**

从名字上可以看出，它既是 PropertySource的集合，多个PropertySource构成PropertySources。

```java
public interface PropertySources extends Iterable<PropertySource<?>> {
    //是否含有名字为name的PropertySource
    boolean contains(String name);
    //获取名字为name的PropertySource
    PropertySource<?> get(String name);
}
```

**MutablePropertySources**

 从名字上看，它表示一个可变的PropertySource集合，即可以随时增加或者移除 PropertySource。内部使用CopyOnWriteArrayList容器来存放：

```java
private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();
```



